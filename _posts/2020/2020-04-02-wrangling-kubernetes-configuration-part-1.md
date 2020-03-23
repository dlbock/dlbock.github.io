---
title: Wrangling Kubernetes configuration (Part 1)
share: true
tags:
  - helm charts
  - jsonnet
  - kubernetes configuration management
---

I've recently been working a lot with helm charts and Kubernetes configuration and one of the challenges has been managing the differences between all the installation methods and ensuring it is deployable on multiple Kubernetes platforms e.g. helm chart on [Helm Hub](https://hub.helm.sh/), helm chart on the [Rancher Library Catalog](https://rancher.com/docs/rancher/v2.x/en/catalog/built-in/), single YAML file format for both Kubernetes and OpenShift, a [Google Cloud Platform Marketplace](https://console.cloud.google.com/marketplace) application just to name a few.

The underlying Kubernetes resources that need to be created are not _too_ different from one platform to another, but there was enough difference for a fair amount of complexity not to mention the fact that they had to be published/pushed to various locations. There are [many tools](https://blog.argoproj.io/the-state-of-kubernetes-configuration-management-d8b06c1205) out there for Kubernetes configuration management but they aren't always a one-size-fits-all solution for your needs.

A few things things that we started doing in attempts to manage the increasing complexity:
1. Create a single canonical source where the resources can be generated from
2. Use [Jsonnet](https://jsonnet.org/) templates where necessary
3. Use `helm template` to generate configuration in single YAML file format

I'll illustrate the second point a bit more with a simple example.

Consider a snippet of the following `Chart.yaml` for `My Awesome Application`'s helm chart for Helm Hub.

```yaml
apiVersion: v1
name: my-awesome-application
version: 1.0.26
appVersion: 1.1
description: My Awesome Application
home: https://www.example.com/
icon: https://remote-site.com/my-icon.png
...
```

Compare that with a snippet of this other `Chart.yaml` for `My Awesome Application`'s helm chart for the Rancher Library Catalog.

```yaml
apiVersion: v1
name: my-awesome-application
version: 1.0.26
appVersion: 1.1
description: My Awesome Application
home: https://www.example.com/
icon: file://../my-icon.png
...
```

The difference is the `icon` path points to a local file for the Rancher version to provide support for air-gapped users, whereas the Helm Hub version points to a remote file. This simple difference could be handled in a few ways:

1. Python (or bash, etc) script to swap out the icon path one of the `Chart.yaml` versions
2. Maintain 2 versions of the same file
3. Use a jsonnet template to manage the difference

Option #1 got a little gnarly using `awk` as there were slight differences between Linux distributions of GNU `awk` (for running on the CI machine) and OS X `awk` (for local development). The `bash` gurus out there might know of a different tool/way to resolve this but I had to find an alternative as I wasn't one.

Option #2 got really annoying over time as everytime someone made any changes, they had to remember to increase the Chart `version` in _two_ separate files.

So we went with Option #3 using the following `Chart.jsonnet` template:

```json
local type = std.extVar('type');
local localIcon = "file://../my-icon.png";
local remoteIcon = "https://remote-site.com/my-icon.png";
local iconPath = if type == 'rancher' then localIcon else remoteIcon;

{
  "apiVersion": "v1",
  "name": "my-awesome-application",
  "version": "1.0.26",
  "appVersion": 1.1,
  "description": "My Awesome Application",
  "home": "https://www.example.com/",
  "icon": iconPath
}
```

Then run `jsonnet` with the aforementioned template:

```bash
# helm
jsonnet --ext-str type=helm -o $TARGET_DIR/Chart.json Chart.jsonnet

# rancher
jsonnet --ext-str type=rancher -o $TARGET_DIR/Chart.json Chart.jsonnet
```

Note that `jsonnet` generates `json` files but you can use `python` to convert that to `yaml` fairly easily and then delete the generated `json` file if you no longer need it:

```python
#!/usr/bin/env python3

import sys, yaml, json

yaml.safe_dump(json.load(sys.stdin), sys.stdout, default_flow_style=False)

```

What I like about using a `jsonnet` template in this scenario:
- It's readable and the differences are encoded in the same location as the contents of the file itself so there isn't some external script that modifies the file after the fact
- It's deterministic and not system dependent
- And of course, no duplication necessary

Next time, we'll take a look at another (hopefully slightly more complex) example of Kubernetes configuration wrangling.
