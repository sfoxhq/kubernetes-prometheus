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

## Manifest Outputs

Cleanup is straightforward but some resources may persist since they aren't namespaced.
The list of outputs is included for convenience of auditing.

```
namespace "monitoring" created
clusterrolebinding.rbac.authorization.k8s.io "prometheus" created
clusterrole.rbac.authorization.k8s.io "prometheus" created
serviceaccount "prometheus-k8s" created
configmap "alertmanager-templates" created
configmap "alertmanager" created
deployment.extensions "alertmanager" created
service "alertmanager" created
deployment.extensions "grafana-core" created
configmap "grafana-import-dashboards" created
job.batch "grafana-import-dashboards" created
secret "grafana" created
service "grafana" created
configmap "prometheus-core" created
deployment.extensions "prometheus-core" created
deployment.extensions "kube-state-metrics" created
clusterrolebinding.rbac.authorization.k8s.io "kube-state-metrics" created
clusterrole.rbac.authorization.k8s.io "kube-state-metrics" created
serviceaccount "kube-state-metrics" created
service "kube-state-metrics" created
daemonset.extensions "node-directory-size-metrics" created
daemonset.extensions "prometheus-node-exporter" created
service "prometheus-node-exporter" created
configmap "prometheus-rules" created
service "prometheus" created
```

## Credit

Alertmanager configs and integration in this repository was heavily inspired by the implementation in [kayrus/prometheus-kubernetes](https://github.com/kayrus/prometheus-kubernetes).
