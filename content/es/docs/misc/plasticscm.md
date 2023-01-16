---
title: "PlasticSCM"
description: "PlasticSCM extensions and solutions."
lead: "PlasticSCM extensions and solutions."
date: 2020-10-13T15:21:01+02:00
lastmod: 2020-10-13T15:21:01+02:00
draft: false
images: []
menu:
  docs:
    parent: "misc"
weight: 999
toc: true
---

## Download Plastic

[Official downloads](https://www.plasticscm.com/download).

## PlasticSCM Github Extension

We have created an extension to access your Github Issues, directly from PlasticSCM:

* Available for Linux and Windows.
* [Open source](https://github.com/equilaterus-gamestudios/PlasticSCM-GitHub-extension).

## PlasticSCM common errors

- **Connection refused during check-in (Linux)**: execute the following commands

  ```
  sudo systemctl stop plasticscm-server
  sudo systemctl start plasticscm-server
  ```

## Plastic custom file types (filetypes.conf)

> Plastic SCM handles two different file types: binaries and text files. By default, Plastic tries to identify a newly added file by using an internal algorithm and a built-in list of known extensions. But sometimes a file that should be binary is identified as text or viceversa. [Source](https://blog.plasticscm.com/2008/03/custom-file-types.html).

Create a file named **filetypes.conf** under the root folder of your workspace, and customize the contents based on the following snippet:

```bash
# PlasticSCM custom file types.
# Syntax: <extension>:<type>
# Type can be 'txt' or 'bin'.
# Examples:
#     .cpp:txt
#     .jpg:bin
.uplugin:txt
.uproject:txt
```

PlasticSCM filetypes.conf example.
