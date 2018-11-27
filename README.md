# Kubernetes Setup for Prometheus and Grafana

## SFOX Prerequisite
Our in-house version places prometheus server onto a dedicated monitoring node in
our kubernetes clusters. The node is automated via a kops instance group as
follows:

```
apiVersion: kops/v1alpha2
kind: InstanceGroup
metadata:
  labels:
    kops.k8s.io/cluster: kops-test-us-east-1.k8s.local
  name: monitoring
spec:
  image: ami-43a15f3e
  machineType: t2.medium
  maxSize: 1
  minSize: 1
  nodeLabels:
    kops.k8s.io/instancegroup: monitoring
  role: Node
  subnets:
  - us-east-1a
  taints:
  - monitoring=prometheus:NoSchedule
```

Ensure the instance group is created, then ensure `nodeLabels` and `taints` properties
conform to the example per cluster.

## Quick start

To quickly start all the things just do this:
```bash
kubectl apply \
  --filename https://raw.githubusercontent.com/giantswarm/kubernetes-prometheus/master/manifests-all.yaml
```

This will create the namespace `monitoring` and bring up all components in there.

To shut down all components again you can just delete that namespace:
```bash
kubectl delete namespace monitoring
```

## Default Dashboards

If you want to re-import the default dashboards from this setup run this job:
```bash
kubectl apply --filename ./manifests/grafana/import-dashboards/job.yaml
```

In case the job already exists from an earlier run, delete it before:
```bash
kubectl --namespace monitoring delete job grafana-import-dashboards
```

To access grafana you can use port forward functionality
```bash
export POD_NAME=$(kubectl get pods --namespace monitoring -l "app=grafana,component=core" -o jsonpath="{.items[0].metadata.name}")

kubectl port-forward --namespace monitoring $POD_NAME 3000:3000
```
And you should be able to access grafana on `http://localhost:3000/login`

## More Dashboards

See grafana.net for some example [dashboards](https://grafana.net/dashboards) and [plugins](https://grafana.net/plugins).

- Configure [Prometheus](https://grafana.net/plugins/prometheus) data source for Grafana.<br/>
`Grafana UI / Data Sources / Add data source`
  - `Name`: `prometheus`
  - `Type`: `Prometheus`
  - `Url`: `http://prometheus:9090`
  - `Add`

- Import [Prometheus Stats](https://grafana.net/dashboards/2):<br/>
  `Grafana UI / Dashboards / Import`
  - `Grafana.net Dashboard`: `https://grafana.net/dashboards/2`
  - `Load`
  - `Prometheus`: `prometheus`
  - `Save & Open`

- Import [Kubernetes cluster monitoring](https://grafana.net/dashboards/162):<br/>
  `Grafana UI / Dashboards / Import`
  - `Grafana.net Dashboard`: `https://grafana.net/dashboards/162`
  - `Load`
  - `Prometheus`: `prometheus`
  - `Save & Open`

## What metrics are exposed?
Each component in the system has different metrics you can explore by prefix. Refer
to documentation for each exporter for a comprehensive list of exposed metrics.

**cadvisor**
- `container_`
- `machine_`

**prometheus**
- `go_`
- `http_request_`
- `http_response_`
- `process_`
- `prometheus_`

**kube-state-metrics**: metrics about the state of kubernetes objects
- `kube_`

**prometheus-node-exporter**: host-level metrics
- `node_`
- `process_`

**node-directory-size-metrics**: metrics about disk usage on the nodes
- `node_directory_size_bytes`

## Manifest Outputs

The list of outputs is included for convenient auditing.

```
namespace "monitoring" created
clusterrole.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrole.rbac.authorization.k8s.io "prometheus" created
clusterrolebinding.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrolebinding.rbac.authorization.k8s.io "prometheus" created
configmap "alertmanager" created
configmap "alertmanager-templates" created
configmap "grafana-import-dashboards" created
configmap "prometheus-core" created
configmap "prometheus-rules" created
daemonset.extensions "node-directory-size-metrics" created
daemonset.extensions "prometheus-node-exporter" created
deployment.extensions "alertmanager" created
deployment.extensions "grafana-core" created
deployment.extensions "kube-state-metrics" created
deployment.extensions "prometheus-core" created
job.batch "grafana-import-dashboards" created
persistentvolumeclaim "prometheus-volumeclaim" unchanged
secret "grafana" created
service "alertmanager" created
service "grafana" created
service "kube-state-metrics" created
service "prometheus" created
service "prometheus-node-exporter" created
serviceaccount "kube-state-metrics" created
serviceaccount "prometheus-k8s" created
storageclass.storage.k8s.io "io1-retain-us-east-1a" created
```

These resources persist after cleanup since they aren't namespaced. Remove them manually.

```
clusterrole.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrole.rbac.authorization.k8s.io "prometheus" created
clusterrolebinding.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrolebinding.rbac.authorization.k8s.io "prometheus" created
```

## Credit

Alertmanager configs and integration in this repository was heavily inspired by the implementation in [kayrus/prometheus-kubernetes](https://github.com/kayrus/prometheus-kubernetes).
