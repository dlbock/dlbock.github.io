---
layout: post
title: Learnings from operating Concourse for the past year
tags:
  - concourse
  - cloudsql
  - kubernetes
---

I have had to operate a [Concourse](https://concourse-ci.org/) cluster on GKE for the last year or so. There have been so many things that I wish I knew back when I started that I have learned over the last few months. So, this is mostly a reminder to my future self in the event that the information is helpful, but maybe this will also help someone else who is going through a similar journey.

## Use a managed database

It's possible that I'm missing many skills in this area, but having to manage a PostgreSQL database that's installed via a helm chart dependency, running via Kubernetes Pod with the data in a Persistent Volume is a giant PITA. You have to hand-roll your own backup & restore strategy, you have to manually scale up the Persistent Volume size. It's one less headache if you can use a managed database like Google CloudSQL from the get-go. Backups and sizing are automatic, and you can create & manage the resource via Terraform. I haven't had to go through a PostgreSQL version update yet with CloudSQL, so that story remains to be told.

## Enable Concourse to emit metrics

Concourse can be configured to [emit metrics](https://concourse-ci.org/metrics.html) about its system health and the builds that it is running. This information is key to understanding how your Concourse cluster is doing, and where the actual & potential problems are. There are quite a few Metric Emitters available, Prometheus being one of them. If you're installing Concourse on Kubernetes via its helm chart, you can see what those options are in the [values.yaml](https://github.com/concourse/concourse-chart/blob/master/values.yaml) (search for `metrics:` and look at the available metric emitter sections below that).

Another thing to point out here is that if you want to implement autoscaling of the Concourse workers, you will need these metrics if you want to use anything other than the pod's CPU & Memory usage to configure the Horizontal Pod Autoscaler.

## Use SSDs for Concourse Worker Persistent Volumes

If you are running Concourse in a Kubernetes cluster, and installing the Concourse workers as a StatefulSet instead of a Deployment, ensure that the Persistent Volumes are using a StorageClass with type `pd-ssd`. It's configurable in the Concourse helm chart. For builds that are I/O heavy will run into issues very quickly if the volumes are using a `pd-standard` StorageClass.

## Configure the appropriate Container Placement Strategy

You will inevitably need to configure a [container placement strategy](https://concourse-ci.org/container-placement.html) that works for you. The default set in the helm chart is `volume-locality`, which could mean that a handful of workers will be overloaded because that's where the their inputs ended up. I recommend understanding all the available strategies and pick one (or more) that works for you.

We have been using the `limit-active-tasks` strategy for the past year, with the max allowed active build tasks per worker `limitActiveTasks` set to `5`. That number is probably too small, but we were trying to be conservative when we encountered the issue of some workers being overloaded with the default `volume-locality` strategy.

There is now an option to [chain more than one container placement strategies together](https://concourse-ci.org/container-placement.html#chaining-placement-strategies), which wasn't available last year. We will probably be switching to a combination of `limit-active-tasks` + `volume-locality` in the very near future.

## Set an acceptable termination grace period

When [workers are restarted/deleted](https://github.com/concourse/concourse-chart#restarting-workers) (either manually or by Kubernetes), we can configure the `terminationGracePeriodSeconds` value to provide an upper limit to how long Kubernetes will wait for Concourse to gracefully [retire](https://concourse-ci.org/internals.html#RETIRING-table) the worker before forcefully terminating the container.

This is a tricky number to set as it depends on the builds that are running in your cluster and the average/max time a build takes to complete, and how comfortable you are with the possibility of workers being killed before its tasks are drained completely. This number is also important in the event that you implement worker autoscaling as you probably want Kubernetes to scale down workers only after they've been retired properly.
