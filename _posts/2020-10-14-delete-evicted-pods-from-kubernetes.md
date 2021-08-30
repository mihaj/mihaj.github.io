---
layout: post
title: Delete evicted pods from Kubernetes
excerpt_separator: <!--more-->
author: Miha J.
tags: kubernetes
---
<!--more-->
In the previous nugget I covered listing evicted pods in Kubernetes. Like with all CRUD `kubectl` operations you can also delete those by replacing `get` with `delete`.

```powershell
kubectl delete pods --all-namespaces --field-selector 'status.phase==Failed'
```