Command:
```shell
# STEP 1.
# Generate a Private Key
openssl genrsa -out shiv.key 2048

# Create a Certificate Signing Request
openssl req -new -key shiv.key -out shiv.csr -subj "/CN=shiv"
# Create a Kubernetes CertificateSigningRequest
# First encode the CSR:
openssl req -in shiv.csr -noout -subject

# STEP 2
# Generate Base64 as ONE LINE
base64 -w 0 shiv.csr > shiv.csr.b64

# Check the whitespace and store in file.
base64 < shiv.csr | tr -d '\n' > shiv.csr.b64

# STEP 3
# create the YAML for you
CSR=$(cat shiv.csr.b64)

cat > shiv-csr.yaml <<EOF
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: shiv
spec:
  request: ${CSR}
  signerName: kubernetes.io/kube-apiserver-client
  usages:
    - client auth
EOF

# STEP 4
# Validate the Base64 before Kubernetes
base64 -d shiv.csr.b64 > /tmp/test.csr

# Then
openssl req -in /tmp/test.csr -noout -subject

# STEP 5
# apply the kubernetes yaml file
kubectl apply -f shiv-csr.yaml

# Get 
kubectl get csr

# You should see something like:
NAME   AGE   SIGNERNAME                                    REQUESTOR   CONDITION
shiv   5s    kubernetes.io/kube-apiserver-client           ...         Pending

# Then after Approve the user
kubectl certificate approve shiv

# Then after Get the signed certification 
kubectl get csr shiv -o jsonpath='{.status.certificate}' | base64 --decode > shiv.crt

# add the user to kubeconfig 
kubectl config set-credentials shiv \
  --client-certificate=shiv.crt \
  --client-key=shiv.key
  
# Add the user context
kubectl config set-context shiv-context \
  --cluster=<cluster-name> \
  --user=shiv
  
# Switch the user 
kubectl config use-context shiv-context

# final check the current user
kubectl auth whoami
```

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