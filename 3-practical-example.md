# Practical example: deploy Prometheus on Kubernetes
- Written on 19/1/6
- Unpublished
- Updated never
## By Leland Later

[Kubernetes](https://github.com/kubernetes/kubernetes) and [Prometheus](https://github.com/prometheus/prometehues) are the first two projects to join the [Cloud Native Computing Foundation (CNCF)](https://www.cncf.io/). Unsurprisingly, there are vibrant projects available to install Prometheus on Kubernetes and have that Prometheus monitor the Kubernetes on which it runs. At Houseparty, we the CoreOS project [Prometheus Operator](https://github.com/coreos/prometheus-operator) to solve operational challenges in application and infrastructure monitoring. Incidentally, our usage of Jsonnet increased with our usage of Prometheus Operator.

> Note: Kubernetes, Prometheus and the Prometheus Operator are sophisticated projects with a steep learning curve. This post will not attempt to explain these projects in detail.

Prometheus Operator defines several [CustomResourceDefinitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions), or Kubernetes API extensions, for `Prometheus`, `PrometheusRule`, `ServiceMonitor` and `Alertmanager`. These resources are treated like any another Kubernetes API object. In Prometheus Operator source code, [they are defined in Jsonnet](https://github.com/coreos/prometheus-operator/tree/master/jsonnet/prometheus-operator).

In a previous post, 

#### Tags
- Kubernetes
- Jsonnet