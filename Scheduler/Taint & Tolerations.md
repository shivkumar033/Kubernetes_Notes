## 1. What is Taint?

A **Taint** is applied to a **Node** to prevent Pods from being scheduled on it unless they have a matching toleration.

> **Taint = Node blocks Pods 🚫**

Example:
```bash
kubectl taint nodes minikube-m02 gpu=true:NoSchedule
```
Now normal Pods cannot be scheduled on `node1`.

---
## 2. What is Toleration?

A **Toleration** is added to a **Pod** to allow it to run on a Node with a matching taint.

> **Toleration = Pod has permission 🎫**
### YAML Example
```bash
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod

spec:
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"

  containers:
    - name: nginx
      image: nginx
```

This Pod can tolerate:
```bash
Node taint:
gpu=true:NoSchedule
```

## 3. Complete Example

#### Step 1 — Taint the Node
```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

```bash
node1 🚫 gpu=true:NoSchedule
```
#### Step 2 — Normal Pod
```bash
Normal Pod
    ↓
node1 🚫
    ↓
❌ Cannot schedule
```
#### Step 3 — Pod with Toleration
```bash
GPU Pod
 🎫 gpu=true
    ↓
node1 🚫 gpu=true
    ↓
✅ Allowed
```

## 4. Three Taint Effects

|Effect|Meaning|
|---|---|
|`NoSchedule`|Don't schedule new Pods without toleration|
|`PreferNoSchedule`|Try to avoid scheduling Pods|
|`NoExecute`|Don't schedule + remove existing non-tolerating Pods|

Example:
```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

### 5. Useful Commands
```bash
# View node taints
kubectl describe node node1

# Add taint
kubectl taint nodes node1 gpu=true:NoSchedule

# Remove taint
kubectl taint nodes node1 gpu=true:NoSchedule-

# Check nodes
kubectl get nodes
```