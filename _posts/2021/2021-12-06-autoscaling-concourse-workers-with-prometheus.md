---
layout: post
title: Autoscaling Concourse workers with custom Prometheus metrics
tags:
  - concourse
  - prometheus
  - kubernetes
---

If you're operating a [Concourse](https://concourse-ci.org/) cluster on Kubernetes, you may or may not need to implement the autoscaling of Concourse workers to automatically handle expanding and contracting workloads. The Concourse [helm chart](https://github.com/concourse/concourse-chart/blob/master/values.yaml) supports using Kubernetes' [Horizontal Pod Autoscaler](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) to enable autoscaling based on observed CPU utilization or custom metrics.

By default, each Concourse worker allows only 250 containers to run concurrently. When a worker reaches its max of 250 concurrent running containers, it is no longer able to take on additional tasks and when all of your workers reach that point, then your Concourse cluster is basically unusable. You could [increase that limit](https://www.engineerbetter.com/blog/increasing-concourse-container-limits/) but you should understand the implications before actually doing that. The other option would be to autoscale the Concourse workers based on the average number of concurrent running containers per worker.

Basically what we need is in the following order:
- Expose Prometheus metrics from Concourse
- Install Prometheus to collect metrics
- Install the Prometheus Adapter to act as a metric API server to make custom metrics available to Kubernetes
- Enable autoscaling on the Concourse side, which will create a `HorizontalPodAutoscaler` resource and utilize the custom metric made available to Kubernetes to autoscale the Concourse worker pods

There isn't a lot of documentation out there specific to this use case (or at least I couldn't find it), so hopefully this will be belpful to someone out there (or future me). 

## Enable Concourse to expose Prometheus metrics

I mentioned in a [previous post](https://dlbock.github.io/2021/10/15/operating-concourse-learnings.html) that Concourse can be enabled to emit metrics about itself. It supports a few types of metric emitters, but I chose Prometheus since we had the most experience with it.

## Install the Prometheus server

Once you've enabled your Concourse cluster to emit Prometheus metrics, if you already have a Prometheus server running in the same Kubernetes cluster, it'll automatically find and collect those metrics. We installed Prometheus via its [helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus) and the only component we needed here is the Prometheus `server` that will pull the metrics exposed by your services via the `/metrics` endpoints and store them in the time-series database for querying.

Here's the `values.yaml` configuration we used for the Prometheus helm chart:

```
alertmanager:
  enabled: false

kubeStateMetrics:
  enabled: false

nodeExporter:
  enabled: false

pushgateway:
  enabled: false

server:
  nodeSelector:
    compute-load: prometheus
  persistentVolume:
    size: 20Gi
    storageClass: "${server_storage_class_name}"
  resources:
    requests:
      cpu: 2
      memory: 3Gi
  retention: "7d"
```

## Install the Prometheus adapter

In order to make custom metrics available to Kubernetes, we need them to be exposed via Kubernetes' custom metrics API. This is enabled via "adapter" API servers like the [Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter). We installed the Prometheus Adapter via its [helm chart](https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-adapter) and here is the `values.yaml` configuration we used:

```
prometheus:
  url: http://<name-of-prometheus-k8s-service>.<namespace>.svc.cluster.local
  port: 80

resources:
  requests:
    cpu: 200m
    memory: 256Mi
  limits:
    cpu: 500m
    memory: 512Mi

rules:
  default: false
  custom:
    - seriesQuery: 'concourse_workers_containers{worker=~"concourse-worker-.*", kubernetes_namespace="concourse"}'
      resources:
        overrides:
          kubernetes_namespace: { resource: "namespace" }
          worker: { resource: "pod" }
      name:
        matches: "^(.*)"
        as: "${1}_avg"
      metricsQuery: 'avg_over_time(<<.Series>>{<<.LabelMatchers>>,worker=~"concourse-worker-.*"}[5m])'

```

This is where things got confusing and fuzzy for me to understand what exactly Kubernetes is expecting with this custom metric and I found the following references to be the most helpful:

- [A very extensive walkthrough of the configuration of the Prometheus Adapter](https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/config-walkthrough.md)
- [The Prometheus Adapter configuration reference](https://github.com/kubernetes-sigs/prometheus-adapter/blob/master/docs/config.md) which is also very helpful
- [And of course the official Kubernetes docs regarding Horizontal Pod Autoscaling and how it works](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/). Personally the following paragraph was key to helping me understand what it was expecting in what resulted from the configured `metricsQuery`:

```
For per-pod resource metrics (like CPU), the controller fetches the metrics from the resource metrics API
for each Pod targeted by the HorizontalPodAutoscaler. Then, if a target utilization value is set, the controller
calculates the utilization value as a percentage of the equivalent resource request on the containers in each
Pod. If a target raw value is set, the raw metric values are used directly. The controller then takes the mean
of the utilization or the raw value (depending on the type of target specified) across all targeted Pods, and
produces a ratio used to scale the number of desired replicas.

For per-pod custom metrics, the controller functions similarly to per-pod resource metrics, except that it
works with raw values, not utilization values.
```

## Enable autoscaling in Concourse

Now you're finally ready to configure Concourse to create the `HorizontalPodAutoscaler` resource using the resulting `concourse_workers_containers_avg` metric. Since we installed Concourse via its [helm chart](https://github.com/concourse/concourse-chart), we just had to edit the `values.yaml` to add the following section:

```
concourse:
  ...
  worker:
    ...
    autoscaling:
      maxReplicas: 30
      minReplicas: 24
      customMetrics:
        - type: Pods
          pods:
            metric:
              name: concourse_workers_containers_avg
            target:
              type: AverageValue
              averageValue: 180
```

This basically creates a `HorizontalPodAutoscaler` (HPA) resource that would have a minimum of 24 Concourse worker pods, and if the average value of `concourse_workers_containers_avg` goes above 180 using the [built-in scaling algorithm](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/#algorithm-details), it will slowly scale up the number of pods to accommodate the load. Since the maximum replicas is configured at 30, there will never be more than 30 Concourse worker pods running with this HPA configuration.

## A few additional things to note

- Set resource (cpu/memory) requests on your Concourse workers so that each worker pod knows how much resources it can reasonably use. If you're using managed Kubernetes services like Google Kubernetes Engine (GKE), you should also configure autoscaling on the node pool that runs Concourse so that it can scale the nodes as more worker pods are added/removed.
- You should set a reasonable value for `terminationGracePeriodSeconds` when configuring the Concourse workers as this will tell Kubernetes how long to wait to allow the workers to drain current tasks and retire themselves before terminating it. If you have pipelines that run long running jobs, you might have to sequester them to separate worker groups and opt them out of auto-scaling as we don't want Kubernetes to terminate worker pods midway through running tasks.
