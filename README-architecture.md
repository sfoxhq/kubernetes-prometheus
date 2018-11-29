# Architecture 

![prometheus architecture](https://prometheus.io/assets/architecture.png)

---

## Implementation

#### Prometheus Server

NODE SSD (metrics persistence): needs a pv, pvc, statefulset 

PROMETHEUS SERVER: manifests/prometheus/deployment.yaml

PROMETHEUS SERVER CONFIG: manifests/prometheus/configmap.yaml

#### Service Discovery 

AUTO SERVICE DISCOVERY: scrape configs + kubernetes_sd
- see https://github.com/prometheus/prometheus/blob/release-2.4/documentation/examples/prometheus-kubernetes.yml
- scrape configs 
  - job_name: 'kubernetes-nodes'
  - job_name: 'kubernetes-endpoints'
  - job_name: 'kubernetes-services'
  - job_name: 'kubernetes-pods'

#### Prometheus Alerting

PROMETHEUS ALERTING: manifests/alertmanager/deployment.yaml

PROMETHEUS ALERTING RULES: manifests/prometheus/prometheus-rules.yaml

PROMETHEUS ALERTING CONFIG: manifests/alertmanager/configmap.yaml
- notifications
  - route (slack)
  - receiver (slack)

#### Data Visualization and Export

GRAFANA: manifests/grafana/deployment.yaml

GRAFANA PROMETHEUS DATASOURCE: manifests/grafana/import-dashboards/configmap.yaml
- prometheus-datasource.json

GRAFANA DASHBOARD: manifests/grafana/import-dashboards/configmap.yaml
- grafana-net-2-dashboard.json
- grafana-net-737-dashboard.json

GRAFANA DATASOURCE AND DASHBOARD IMPORT JOB: manifests/grafana/import-dashboards/job.yaml

#### Prometheus Targets & Exporters 
EXPORTERS: libraries and servers which export metrics from third-party systems as Prometheus metrics

1. [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics): /manifests/prometheus/kube-state-metrics/deployment.yaml
  - listens to the Kubernetes API server and generates metrics about the state of the objects
    - Deployment
    - Pod
    - Service
    - etc

2. [node-directory-size-metrics](https://github.com/giantswarm/kubernetes-prometheus/blob/master/manifests/prometheus/node-directory-size-metrics/daemonset.yaml): /manifests/prometheus/node-directory-size-metrics/daemonset.yaml
  - metrics about disk usage on the nodes

3. [node-exporter](https://github.com/prometheus/node_exporter): /manifests/prometheus/node-exporter/daemonset.yaml
  - host-level metrics (hardware and os metrics exposed by unix kernels)
    - cpu
    - memory 
    - network

