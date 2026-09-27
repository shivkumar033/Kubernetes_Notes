### Introduction

Prometheus is an open-source monitoring system widely used to collect and store time-series metrics from Kubernetes clusters, applications, and infrastructure.

### PART 1 — What is Prometheus?

Prometheus is a pull-based monitoring system that collects metrics from:

- Kubernetes components
- Nodes
- Pods
- Applications
- Exporters

It stores data in time-series format and provides PromQL for querying metrics.

### PART 2 — Install Prometheus on Kubernetes using Helm

**Step 1 — Create Namespace**

```shell
kubectl create namespace monitoring
```

**Step 2 — Add Prometheus Helm Repository**

```bash
# Add prometheus in the helm packages
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
```

**Step 3 — Install `kube-prometheus-stack`**

This installs:

- Prometheus
- Alertmanager
- Grafana
- Node Exporter
- Kube-State-Metrics

```bash
# Install prometheus
helm install prometheus prometheus-community/kube-prometheus-stack -n monitoring
```

### PART 3 — Verify Installation

List all pods in the `monitoring` namespace:

```bash
kubectl get pods -n monitoring
```

Expected pods:

- `prometheus-kube-prometheus-prometheus-0`
- `prometheus-kube-prometheus-operator-xxxx`
- `prometheus-grafana-xxxx`

### PART 4 — How Prometheus Scrapes Kubernetes Metrics

Prometheus automatically discovers Kubernetes components through:

- `ServiceMonitor`
- `PodMonitor`
- Node Exporter
- `kube-state-metrics`
- API server
- Etcd
- Kubelet

No manual configuration is needed because `kube-prometheus-stack` provides default scraping configs.

### PART 5 — Access Prometheus Dashboard (Port Forwarding)

Use this command to port-forward Prometheus:

```shell
# Using port-forward to access dashboard
kubectl port-forward --address 0.0.0.0 -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# Acess dashboard using expose prometheus services
kubectl expose service prometheus-server --type=NodePort --target-port=9090 --name=promethus-server-ext -n monitoring
```

Then open Prometheus in your browser:

```
http://<your-node-ip>:9090
```

### PART 6 — PromQL Queries

**CPU usage (all containers)**

```
container_cpu_usage_seconds_total
```

**Node CPU usage**

```
rate(node_cpu_seconds_total{mode!="idle"}[5m])
```

**Pod CPU usage**

```
rate(container_cpu_usage_seconds_total{image!=""}[5m])
```

**Node memory used**

```
node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes
```

**Kubelet health**

```
up{job="kubelet"}
```

### PART 7 — Install and Access Grafana Dashboard

**Step 1 — Install Grafana using helm**

```bash
helm repo add grafana https://grafana.github.io/helm-charts

helm repo update

helm install grafana grafana/grafana
```

**Step 2 — Port Forward Grafana**

```shell
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80

# Acess dashboard using expose grafana services
kubectl expose service grafana --type=NodePort --target-port=3000 --name=grafana-ext -n monitoring
```

Open Grafana in your browser:

```
http://<your-node-ip>:3000
```

**Step 3 — Get Grafana Admin Password**

```shell
kubectl get secret -n monitoring prometheus-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
```

**Step 4 — Built-in Dashboards**

Inside Grafana → Dashboards → Browse, you will find dashboards such as:

- `Kubernetes / Compute Resources / Cluster`
- `Kubernetes / API Server`
- `Nodes`
- `Pods`

These are great for visualization and YouTube demos.

### PART 8 — Cleanup

```shell
helm uninstall prometheus -n monitoring
kubectl delete ns monitoring
```