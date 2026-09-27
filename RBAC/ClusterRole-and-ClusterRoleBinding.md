## 1. ClusterRole

A **ClusterRole** defines permissions at the **cluster level**.

Example: allow a user to view Pods in node level:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
  - apiGroups: [""]
    resources: ["node"]
    verbs: ["get", "list", "watch"]
```

```shell
# Create it 
kubeclt apply -f clusterrole.yaml

# Check it
kubectl get clusterrole
```

#### Creating ClusterRole Using `Kubectl` Command
```shell
kubectl create clusterrole node-reader --verb=list,get,watch --resource=node

# See Details about clusterroles
kubectl describe clusterrole/node-reader
```

---

## 2. ClusterRoleBinding

A **ClusterRoleBinding** connects a **user** to a **ClusterRole**.

For your `shiv` user:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: shiv-pod-reader
subjects:
  - kind: User
    name: shiv
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply:
```shell
kubectl apply -f clusterrolebinding.yaml
```
#### Creating ClusterRoleBeinding Using `Kubectl` Command
```shell
kubectl create clusterrolebinding user-binding --clusterrole=node-reader --user=shiv

# checking is created or not
kubectl get clusterrolebinding
```