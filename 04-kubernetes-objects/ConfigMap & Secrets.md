**ConfigMap** stores **non-sensitive configuration data** separately from the application/container.
### Real-world example

Suppose your Node.js application needs:
```bash
APP_NAME = prod-app
APP_PORT = 5000
ENVIRONMENT = production
```

You can create `configmap.yaml` file:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  APP_NAME: prod-app
  APP_PORT: "5000"
  ENVIRONMENT: production
```

Then:
```bash
kubectl apply -f configmap.yaml

# Check it
kubectl get cm

# describe
kubectl describe cm app-config

# Create a ConfigMap Using Command
kubectl create configmap app-config \
  --from-literal=APP_NAME=prod-app \
  --from-literal=APP_PORT=5000
```
---
# How does a Pod use ConfigMap?

There are three common ways.
### 1. ConfigMap → Use Environment Variables in Pod

Example Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: nginx
      env:
        - name: APP_NAME
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_NAME

        - name: APP_PORT
          valueFrom:
            configMapKeyRef:
              name: app-config
              key: APP_PORT
```

## 2. Pod — Inject ALL ConfigMap keys

Use `envFrom`:
```
apiVersion: v1
kind: Pod
metadata:
  name: dairy-app
spec:
  containers:
    - name: app
      image: nginx
      envFrom:
        - configMapRef:
            name: app-config
```

Now Kubernetes automatically creates these environment variables inside the container:

## 3. Inject the ConfigMap Key with Volume

Mount ConfigMap data as files inside the container.
Pods automatically update Value when the ConfigMap changes. Don't Restart the pods.

Pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: config-volume
          mountPath: /etc/app

  volumes:
    - name: config-volume
      configMap:
        name: app-config
```

# ConfigMap vs Secret

This is **very important for Kubernetes security**.

|ConfigMap|Secret|
|---|---|
|Non-sensitive configuration|Sensitive information|
|App name|Password|
|Port|API key|
|Environment|Database credentials|
|Feature flags|Tokens|

---
---

## 2. What is a Secret?

A Kubernetes Secret stores sensitive information such as passwords, API keys, tokens, and database credentials.

Think of it like:
- ConfigMap → Non-sensitive configuration.
- Secret → Sensitive configuration.

#### 2. How to create a Secret

 Method 1: Using kubectl command
 ```bash
 kubectl create secret generic db-secret \
  --from-literal=DB_USERNAME=admin \
  --from-literal=DB_PASSWORD=myPassword123
  
# Check it
kubectl get secrets
kubectl describe secret db-secret

# View encoded data:
kubectl get secret db-secret -o yaml
 ```

Method 2: Using YAML
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-secret
type: Opaque
stringData:
  DB_USERNAME: admin
  DB_PASSWORD: myPassword123
```

Apply:
```bash
kubectl apply -f secret.yaml
```

---
### 2. Use Secret in a Pod

#### 1. `valueFrom` — Single key
```yaml
env:
  - name: DB_PASSWORD
    valueFrom:
      secretKeyRef:
        name: db-secret
        key: DB_PASSWORD
```

Injects one Secret key as an environment variable.

#### 2. `envFrom` — All keys
```yaml
envFrom:
  - secretRef:
      name: db-secret
```

Injects all keys from the Secret as environment variables.

#### 3. Volume Mount — As files
```yaml
spec:
  containers:
    - name: app
      image: nginx
      volumeMounts:
        - name: secret-volume
          mountPath: /etc/secrets
          readOnly: true

  volumes:
    - name: secret-volume
      secret:
        secretName: db-secret
```