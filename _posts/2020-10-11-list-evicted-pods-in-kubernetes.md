---
layout: post
title: List evicted pods in Kubernetes
excerpt_separator: <!--more-->
author: Miha J.
tags: kubernetes
---

Sometimes I want to get a list of all evicted pods from Kubernetes. You can do it like this:

```powershell
kubectl get pods --all-namespaces --field-selector 'status.phase==Failed'
```

And the output is:

```
NAMESPACE       NAME                                        READY   STATUS    RESTARTS   AGE
cert-manager    cert-manager-webhook-645b8bdb7-5j5q5        0/1     Evicted   0          2d7h
cert-manager    cert-manager-webhook-645b8bdb7-x9gjh        0/1     Evicted   0          2d7h
ingress-nginx   nginx-ingress-controller-56fc869c98-85tf8   0/1     Evicted   0          142d
service1-dev    service1-dev-5f8956696c-5678z               0/1     Evicted   0          2d8h
service2-dev    service2-v2-dev-5869f86869-n22g5            0/1     Evicted   0          4d3h
```

Nice huh? :)