---
layout: post
title: Delete evicted pods from Kubernetes
excerpt_separator: <!--more-->
author: Miha J.
tags: kubernetes
---

In the previous nugget I covered listing evicted pods in Kubernetes. Like with all CRUD Kubernetes operations you can also delete those by replacing just one keyword.

```powershell
kubectl delete pods --all-namespaces --field-selector 'status.phase==Failed'
```