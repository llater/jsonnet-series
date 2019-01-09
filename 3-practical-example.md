# Practical example: deploy Prometheus on Kubernetes
- Written on 19/1/7
- Unpublished
- Updated never
## By Leland Later

[Kubernetes](https://github.com/kubernetes/kubernetes) and [Prometheus](https://github.com/prometheus/prometheus) are the first two projects to join the [Cloud Native Computing Foundation](https://www.cncf.io/). Unsurprisingly, there are vibrant projects available to install Prometheus on Kubernetes and have that Prometheus monitor the Kubernetes on which it runs. At Houseparty, we the CoreOS project [Prometheus Operator](https://github.com/coreos/prometheus-operator) and its subproject [kube-prometheus](https://github.com/coreos/prometheus-operator/tree/master/contrib/kube-prometheus) to solve operational challenges in application and infrastructure monitoring. Incidentally, our usage of Jsonnet increased with our usage of Prometheus Operator and kube-prometheus.

> Note: Kubernetes, Prometheus and the Prometheus Operator are sophisticated projects with steep learning curves. This post will not attempt to explain these projects in detail.

Prometheus Operator defines several [CustomResourceDefinitions](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/#customresourcedefinitions), or Kubernetes API extensions, for `Prometheus`, `PrometheusRule`, `ServiceMonitor` and `Alertmanager`. These resources are treated like any another Kubernetes API object. In Prometheus Operator source code, [they are defined in Jsonnet](https://github.com/coreos/prometheus-operator/tree/master/jsonnet/prometheus-operator).

kube-prometheus allows for extensibility of the core that Prometheus Operator provides. Prometheus Operator will run your Prometheus and Alertmanager clusters, but to monitor applications on the cluster itself, use kube-prometheus.

To begin working with kube-prometheus, you will need to vendor it with jsonnet-bundler (see this [previous post](https://github.com/llater/jsonnet-series/blob/master/vendoring-with-jb.md) if you have not used jsonnet-bundler).

Run `jb install github.com/coreos/prometheus-operator/contrib/kube-prometheus/jsonnet/kube-prometheus` to get started.

Consider this snippet from `kube-prometheus.jsonnet`:
```
local kp =
(import 'kube-prometheus/kube-prometheus.libsonnet') +
{
  _config+:: {
    namespace: 'monitoring',
    clusterName: std.extVar('cluster'),
    prometheus+:: {
      name: $._config.clusterName,
    },
    alertmanager+:: {
      name: $._config.clusterName,
    },
    prometheus+:: (import 'prometheus.libsonnet').prometheus,
  },
};

{ ['prometheus-' + name]: kp.prometheus[name] for name in std.objectFields(kp.prometheus) }
```

And a file in the same directory, `prometheus.libsonnet`:
```
local prometheusName = std.extVar("name");
local storageClassName = std.extVar("storage");
local env = std.extVar("environment");

local defaultStorageSize = "100Gi";
local defaultMemoryRequest = "2000Mi";
local deafultCpuRequest = "2000m";

{
    _config+:: {
        prometheusName: prometheusName,
        storageClassName: storageClassName,
    },
    prometheus+:: {
        prometheus+: {
            metadata+: {
                name: $._config.prometheusName,
            },
            spec+: {
                storage: {
                    volumeClaimTemplate: {
                        spec: {
                            storageClassName: $._config.storageClassName,
                            resources: {
                                requests: {
                                    storage: if env == "staging" then defaultStorageSize
                                        else if env == "production" then "1000Gi",
                                },
                            },
                        },
                    },
                },
                replicas: 2,
                resources: {
                    requests: {
                        memory: if env == "staging" then defaultMemoryRequest
                            else if env == "production" then "32000Mi" ,
                        cpu: if env == "staging" then deafultCpuRequest
                            else if env == "production" then "4000m",
                    },
                },
            },
        },
    },
}
```

When `kube-prometheus.jsonnet` is compiled, it adds Jsonnet from the vendored `kube-prometheus` code and considers the overrides in the `prometheus.libsonnet`, a file to override the `kube-prometheus` defaults.

Compile the Jsonnet Prometheus Operator deployment with:
```
jsonnet -J vendor -V cluster=app -V name=app-monitoring -V storage=ebs -V environment=staging kube-prometheus.jsonnet > app-monitoring.json
```

Check out the output of `app-monitoring.json`. This JSON contains a complete Kubernetes Prometheus deployment. Look at the block called "prometheus-prometheus":
```
...
"prometheus-prometheus": {
      "apiVersion": "monitoring.coreos.com/v1",
      "kind": "Prometheus",
      "metadata": {
         "labels": {
            "prometheus": "app"
         },
         "name": "app-monitoring",
         "namespace": "monitoring"
      },
      "spec": {
         "alerting": {
            "alertmanagers": [
               {
                  "name": "alertmanager-app",
                  "namespace": "monitoring",
                  "port": "web"
               }
            ]
         },
         "baseImage": "quay.io/prometheus/prometheus",
         "nodeSelector": {
            "beta.kubernetes.io/os": "linux"
         },
         "replicas": 2,
         "resources": {
            "requests": {
               "cpu": "2000m",
               "memory": "2000Mi"
            }
         },
         "ruleSelector": {
            "matchLabels": {
               "prometheus": "app",
               "role": "alert-rules"
            }
         },
         "securityContext": {
            "fsGroup": 2000,
            "runAsNonRoot": true,
            "runAsUser": 1000
         },
         "serviceAccountName": "prometheus-app",
         "serviceMonitorNamespaceSelector": { },
         "serviceMonitorSelector": { },
         "storage": {
            "volumeClaimTemplate": {
               "spec": {
                  "resources": {
                     "requests": {
                        "storage": "100Gi"
                     }
                  },
                  "storageClassName": "ebs"
               }
            }
         },
         "version": "v2.5.0"
      }
   },
...
```

Any of the above fieds may be overriden with custom Jsonnet code. Try compiling with the `-V environment=production` flag and observe how the output changes. Notice how the cluster name `app` labels the Prometheus with a Kubernetes and `app-monitoring` is the name of the Prometheus CRD.

The final JSON output can be applied directly the a Kubernetes API endpoint:
`kubectl apply -f app-monitoring.json`.

Deploying Prometheus on a new or old Kubernetes cluster can be a pain, but Prometheus Operator from CoreOS does the lion's share of this work. Prometheus Operator comes with several alerts preconfigured. Visit the Prometheus UI to begin testing the new monitoring system!

> Hint: if you do not have an ingress controller or load balancer solution in place for the Kubenetes cluster, try the proxy: `kubectl proxy`. Then, visit `127.0.0.1:8001/api/v1/namespaces/monitoring/services/prometheus-<name>:9090/proxy` to check out the Prometheus UI.

### In conclusion
Jsonnet is a great tool but its real power is integration into many modern applications. Jsonnet allows Kubernetes users to deploy supporting services alongside the core app painlessly. We consider monitoring code part of the application deployment. The PrometheusRules and ServiceMonitors are deployed at the same time as our core app and stored with its source code.

#### Tags
- Kubernetes
- Prometheus
- monitoring
- Jsonnet