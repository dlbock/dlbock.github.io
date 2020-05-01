---
layout: post
title: Wrangling Kubernetes configuration (Part 2)
tags:
  - helm charts
  - jsonnet
  - kubernetes configuration management
---

Following up on my [first post]({% post_url /2020/2020-04-02-wrangling-kubernetes-configuration-part-1 %}), where we looked at a simple example of using a Jsonnet template to determine what kind of icon path to use for a `Chart.yaml` file, I'd like to take a look at a less simple, but possibly still contrived, example of utilizing Jsonnet to dynamically reconstruct YAML.

Consider the following `my-application.yaml`:

```yaml
---
apiVersion: v1
kind: Namespace
metadata:
  labels:
    app.kubernetes.io/name: my-application
  name: my-application-namespace
---
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app.kubernetes.io/name: my-application
  name: my-application
  namespace: my-application-namespace
```

I'd like to be able to identify which release of `my-application` is being run by adding an additional label to each resource type:

```yaml
  labels:
    app.kubernetes.io/name: my-application
    app.kubernetes.io/version: 1.0.11
```

Since the version number changes between releases, it is unrealistic for me to have to manually edit this file everytime it changes. So let's take a look at how we can use a Jsonnet template to help us re-generate this YAML file and dynamically add the version label.

`my-application.jsonnet`
```json
local resources = std.parseJson(std.extVar('resources'));
local version = std.extVar('version');

local addVersionToMetadataLabels(resource) = resource + {
	metadata+: { labels+:
	 super.labels + { "app.kubernetes.io/version": version }
	}
};
local resourcesWithVersion = std.map(addVersionToMetadataLabels, resources);

{
  ["namespace.v" + version + ".json"]:
    std.filter(function(res) res.kind == "Namespace", resourcesWithVersion)[0],
  ["serviceaccount.v" + version + ".json"]:
    std.filter(function(res) res.kind == "ServiceAccount", resourcesWithVersion)[0]
}
```

Let's break it down.

```json
local resources = std.parseJson(std.extVar('resources'));
local version = std.extVar('version');
```

This template accepts 2 external parameters `resources` and `version`. Here, `resources` is the contents of the `my-application.yaml` file, which has been converted to JSON format, and `version` is whatever version number you want to use.

```json
local addVersionToMetadataLabels(resource) = resource + {
	metadata+: { labels+:
	 super.labels + { "app.kubernetes.io/version": version }
	}
};
local resourcesWithVersion = std.map(addVersionToMetadataLabels, resources);
```

Next, we have a function `addVersionToMetadataLabels` that takes a resource (in JSON format) and subsequently overrides `metadata.labels` to add an additional label with key `app.kubernetes.io/version` and the provided `version` value.

Then we use the Jsonnet standard library `map` function to apply that to all the JSON resources in `resources`.

Now that we have modified the original contents to include a new additional `app.kubernetes.io/version` label, we can now output the new contents and reconstruct our JSON.

```json
{
  ["namespace.v" + version + ".json"]:
    std.filter(function(res) res.kind == "Namespace", resourcesWithVersion)[0],
  ["serviceaccount.v" + version + ".json"]:
    std.filter(function(res) res.kind == "ServiceAccount", resourcesWithVersion)[0]
}
```

This generates 2 separate JSON files, one for `Namespace` and the other for `ServiceAccount`, and then I have written some bash scripting to convert the JSON back to YAML and shove them back into a single YAML file. The reason I've done this is because Jsonnet outputs JSON in alphabetically order by key, and I needed to preserve the original order of the Kubernetes resources specified in `my-application.yaml`.

Hopefully these few examples illustrates some of the many ways Jsonnet can help manage YAML files and consequently your Kubernetes configuration in a slightly saner way.

I recently gave a talk on this subject at a Women Who Code Austin virtual meetup. I've published the code I've used and the slides on [this repo](https://gitlab.com/dlbock/talks/-/tree/master/wrangling-k8s-config). Feel free to copy and paste whatever makes sense.

If anyone else has other experiences with managing Kubernetes configuration using other tools and would like to share, please do in the comments section below. Would love to learn from you.
