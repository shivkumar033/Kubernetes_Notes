## 1. What are Requests and Limits?

In Kubernetes, resource requests and limits control how much CPU and memory a container can use.

- Requests: The amount of CPU and memory Kubernetes uses to schedule a Pod and reserve capacity for it.
- Limits: The maximum CPU or memory a container is allowed to consume.

Why do we need them?

1. Prevent one application from consuming all node resources.
2. Ensure Kubernetes schedules Pods on nodes with sufficient capacity.
3. Control resource usage in production environments.
4. Reduce the risk of resource contention and application instability.

## 2. YAML syntax

Requests and limits are defined under `resources` inside a container specification.

```yaml
resources:
  requests:
    cpu: "250m"
    memory: "128Mi"
  limits:
    cpu: "500m"
    memory: "256Mi"
```

Explanation:

| Setting                | Meaning                               |
| ---------------------- | ------------------------------------- |
| CPU request `250m`     | Scheduler accounts for 0.25 CPU core  |
| CPU limit `500m`       | Container is limited to 0.5 CPU core  |
| Memory request `128Mi` | Scheduler accounts for 128 MiB        |
| Memory limit `256Mi`   | Container memory is capped at 256 MiB |

Requests are not a hard minimum usage guarantee. A container can use less than its request when it doesn't need the resources.

## 3. example: NGINX Deployment or a Pods

Create a file called `nginx-resources.yaml`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "250m"
              memory: "128Mi"
            limits:
              cpu: "500m"
              memory: "256Mi"
```

### Deploy the application

```bash
kubectl apply -f nginx-resources.yaml
```

Check the Pods:

```bash
kubectl get pods
```

View resource requests and limits:

```bash
kubectl describe pod <pod-name>
```

You will see the resource configuration under the container's `Requests` and `Limits` section.

### Calculate resources for 2 replicas

|Resource|Total request|Total limit|
|---|---|---|
|CPU|500m|1 CPU|
|Memory|256Mi|512Mi|

---
## 4. What happens when limits are exceeded?

CPU: Container is throttled when it tries to exceed its CPU limit. Application may become slow.

Memory: Container can be terminated with `OOMKilled` when it exceeds its memory limit.

## 5. What if resources are insufficient?

If a node has only 2 CPUs available and a Pod requests 3 CPUs, the Pod stays in `Pending` state.

Expected event: `Insufficient cpu`.

## 6. Metrics Server

What is Metrics Server?

Metrics Server collects current CPU and memory usage from Kubernetes Nodes and Pods.

Why needed?

- Check actual resource consumption.    
- Use `kubectl top nodes` and `kubectl top pods`.
- Provide metrics for Horizontal Pod Autoscaler (HPA).

### Is Metrics Server required?

|Feature|Metrics Server required?|
|---|---|
|Configure requests and limits|No|
|Enforce CPU and memory limits|No|
|Use `kubectl top`|Yes|
|HPA based on CPU/memory|Usually yes|

Metrics Server is not responsible for enforcing resource limits. Kubernetes and the container runtime enforce them.

### Enable Metrics Server in Minikube

```bash
minikube addons enable metrics-server
```

### Deploy Metrics Server using YAML

On your Windows PowerShell, you can download it directly on kubernetes official metric server:

```powershell
curl.exe -L `
  https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml `
  -o metrics-server.yaml
```

This downloads the official manifest into your current directory.

### Apply the YAML file

Make sure your Minikube cluster is running:

```bash
minikube status
kubectl get nodes
```

Deploy Metrics Server:

```bash
kubectl apply -f metrics-server.yaml
```

Verify:
```bash
kubectl get pods -n kube-system
kubectl top nodes
kubectl top pods
```