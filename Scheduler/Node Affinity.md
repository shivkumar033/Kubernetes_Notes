**Node Affinity** is used to tell Kubernetes **which Nodes a Pod should or must run on**, based on **Node labels**.
### Simple example

Imagine you have node:
```
Node 1 → CPU
Node 2 → GPU
Node 3 → LPU
```

You want your AI application to run **only on the GPU node**.

You label the node:
```
kubectl label nodes node2 hardware=gpu
```

Then use **Node Affinity** in your Pod:
```
Pod 🧠
  │
  │ "I need hardware=gpu"
  ▼
Node 1 ❌ CPU
Node 2 ✅ GPU
Node 3 ❌ LPU
```

---
## Why use Node Affinity?

Common use cases:
- 🖥️ Run workloads on **GPU nodes**
- 💾 Run databases on nodes with **SSD**
- 🌍 Run workloads in a specific **region/zone**
- 🔒 Run sensitive workloads on **dedicated nodes**
- ⚡ Run high-performance applications on specific hardware
# How to use it

### Step 1: Label a Node
```
kubectl label nodes node1 hardware=gpu
```

Check:
```
kubectl get nodes --show-labels
```

---
### Step 2: Add Node Affinity to Pod Yaml
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod

spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: hardware
                operator: In
                values:
                  - gpu

  containers:
    - name: nginx
      image: nginx
```

The important part is:
```yaml
key: hardware
operator: In
values:
  - gpu
```

It means:
> **Only schedule this Pod on a Node where `hardware=gpu` Labels.**
---
# Two Main Types

### 1. `requiredDuringSchedulingIgnoredDuringExecution`

**Must match.**
```
Pod → Node with matching label ✅
Pod → Node without matching label ❌
```

If no matching node exists, the Pod stays **Pending**.

---
### 2. `preferredDuringSchedulingIgnoredDuringExecution`

**Prefer the matching Node, but it's not mandatory.**
```
Matching Node available → use it 👍
No matching Node → use another Node ✅
```

---
---

## Use both **Node Affinity + Taint/Toleration**
### Simple idea

Imagine a **GPU Node**:
```yaml
GPU Node
├── Taint: gpu=true:NoSchedule 🚫
└── Label: hardware=gpu 🏷️
```

Your Pod needs:
```yaml
Toleration → "I'm allowed on GPU node" 🎫
Affinity    → "I specifically want GPU node" 🎯
```

So:
```yaml
                    GPU Node
              ┌─────────────────┐
              │ Taint: gpu=true │ 🚫
              │ Label: hardware=gpu │
              └────────┬────────┘
                       │
              ┌────────┴────────┐
              │                 │
       Toleration 🎫      Node Affinity 🎯
       "I'm allowed"      "I want this node"
              │                 │
              └────────┬────────┘
                       ▼
                    Pod ✅
```
---
# Step 1: Label the Node

Give the node a label:
```bash
kubectl label nodes node1 hardware=gpu
```

Check:
```bash
kubectl get nodes --show-labels
```
---
# Step 2: Add a Taint

```bash
kubectl taint nodes node1 gpu=true:NoSchedule
```

Now `node1` says:
> 🚫 Only Pods that tolerate `gpu=true` can be scheduled here.

---
# Step 3: Create Pod with Both
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-pod

spec:

  # Toleration
  tolerations:
    - key: "gpu"
      operator: "Equal"
      value: "true"
      effect: "NoSchedule"

  # Node Affinity
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
          - matchExpressions:
              - key: "hardware"
                operator: "In"
                values:
                  - "gpu"

  containers:
    - name: nginx
      image: nginx
```

Apply:
```bash
kubectl apply -f gpu-pod.yaml
```

Check:
```bash
kubectl get pods -o wide
```

You should see the Pod scheduled on the appropriate node.