## 1. Static Pods

A **Static Pod** is a Pod managed directly by the **kubelet** on a specific node, without being managed by the Kubernetes scheduler.

```bash
kubelet
   ↓
/etc/kubernetes/manifests
   ↓
Static Pod
```

### Important directory
```bash
/etc/kubernetes/manifests
```
The kubelet watches this directory and creates/maintains the Pods defined there.

### Example
In Minikube, control-plane Pods such as:

```
etcd-minikube
kube-apiserver-minikube
kube-controller-manager-minikube
kube-scheduler-minikube
```
are typically Static Pods.

### Useful command
```
kubectl get pods -n kube-system
```
### Remember
**Static Pod → kubelet manages it directly**

---
# 2. Manual Scheduling

### What is Manual Scheduling?

Normally:
```
Pod → kube-scheduler → Node
```

With manual scheduling, you tell Kubernetes **exactly which node** should run the Pod.

### YAML
```
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeName: worker-node-1
  containers:
    - name: nginx
      image: nginx
```

Here:
```
nodeName: worker-node-1
```

means:
> Run this Pod on `worker-node-1`.
### Remember
**`nodeName` → directly assign Pod to a node.**

---
# 3. Labels and Selectors

## Labels

Labels are **key-value pairs attached to Kubernetes objects**.

Example:
```
labels:
  app: frontend
  tier: web
```

Think:
```
Pod
 ├── app = frontend
 └── tier = web
```

### See labels
```
kubectl get pods --show-labels
```

---

## Selectors

A selector is used to **find Kubernetes objects based on their labels**.

Example:
```
kubectl get pods --selector tier=web
```

This means:
> Show Pods where `tier=web`.
### Common example

A Service uses a selector to find which Pods should receive traffic:
```
selector:
  app: frontend
```

```
Service
   ↓ selector: app=frontend
   ↓
Pod 1 → app=frontend
Pod 2 → app=frontend
Pod 3 → app=backend ❌
```

### Remember
**Label → identifies an object**
**Selector → finds objects using labels**

---
# 4. Annotations vs Labels

### Labels

Used to **identify and select** resources.
```
labels:
  app: frontend
```

You can use:
```
kubectl get pods -l app=frontend
```

### Annotations

Used to store **additional information/metadata** that Kubernetes doesn't normally use for selecting objects.

Example:
```
annotations:
  description: "Frontend application"
  owner: "development-team"
```

You generally **don't use annotations for selecting Pods**.