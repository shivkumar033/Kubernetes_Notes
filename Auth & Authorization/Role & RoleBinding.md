## Give the user permissions with RBAC
#### step 1. create the roles
Ex: allows `shiv` to read Pods:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

after:
```shell
# apply role yaml
kubectl apply -f roles.yaml
```
---
## Create RoleBinding
#### Step 2. Then connect the user to the Role:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: shiv-pod-reader
subjects:
  - kind: User
    name: shiv
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

after command:
```shell
# Apply 
kubectl apply -f rolebinding.yaml

kubectl auth can-i get pods --as=shiv
# --> yes

kubectl auth can-i delete pods --as=shiv
# --> no
```