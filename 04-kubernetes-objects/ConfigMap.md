**ConfigMap** stores **non-sensitive configuration data** separately from the application/container.

Example:
```bash
APP_NAME=prod-app
APP_PORT=5000
ENV=production
```
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
kubectl get configmap

# describe
kubectl describe configmap app-config

# Add Environment Variable using Command
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

## 2. Pod — use ALL ConfigMap keys

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

## 3. ConfigMap → Use Configuration File in Pod

You can also mount a ConfigMap as a file.

ConfigMap:
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  app.name=prod-app
  app.port=5000
  environment=production
```

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