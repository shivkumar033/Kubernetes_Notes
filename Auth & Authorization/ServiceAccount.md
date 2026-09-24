A ServiceAccount (SA) is a Kubernetes identity used by applications and processes running inside Pods to authenticate to the Kubernetes API.

In simple words:
- Human user: A developer uses `kubectl` to communicate with the Kubernetes API.
- ServiceAccount: An application running inside a Pod uses its ServiceAccount to communicate with the Kubernetes API.

## 1. How to create a ServiceAccount

#### Step 1: Create a ServiceAccount

Create `serviceaccount.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: monitor-sa
```

```bash
# Apply it:
kubectl apply -f serviceaccount.yaml

# Create ServiceAccount using kubectl command:
kubectl create serviceaccount dev-sa

kubectl get serviceaccounts
```

## 2. Create a Role to give permissions

Now we want `monitor-sa` to read Pods, but not create or delete them.

Create `role.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader

rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch"]
```

```bash
# Apply It:
kubectl apply -f role.yaml

# Create role using kubectl command:
kubectl create role pod-reader \
  --verb=get,list,watch \
  --resource=pods \
  -n default
  
# 
```

## 3. Bind the Role to the ServiceAccount

Creating a Role does not automatically give its permissions to a ServiceAccount.
We must connect them using a RoleBinding.

Create `rolebinding.yaml`:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: monitor-binding

subjects:
  - kind: ServiceAccount
    name: monitor-sa

roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: pod-reader
```

```bash
# Apply:
kubectl apply -f rolebinding.yaml

# Create RoleBinding using kubectl command:
kubectl create rolebinding monitor-binding \
  --role=pod-reader \
  --serviceaccount=default:monitor-sa \
  -n default
  
# Test the permissions
kubectl auth can-i list pods \
  --as=system:serviceaccount:default:monitor-sa \
  -n default
  
kubectl auth can-i delete pods \
  --as=system:serviceaccount:default:monitor-sa \
  -n default
  
# List the pods using in ServiceAccount
kubectl get pods --as=system:serviceaccount:default:monitor-sa -n default
```