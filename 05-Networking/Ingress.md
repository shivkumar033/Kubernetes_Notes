**Ingress** is a Kubernetes resource that manages **HTTP/HTTPS traffic coming from outside the cluster** and routes it to different Services.

Instead of exposing every application with a separate LoadBalancer or NodePort, Ingress provides a **single entry point** to route traffic to multiple services.

Example:
```bash
Internet
    │
    ▼
Ingress Controller
    │
    ▼
Ingress Resource
    │
    ▼
Service
    │
    ▼
Pods
```
## Why Use Ingress?

- Expose multiple applications using one IP address (Single Endpoint).
- Perform host-based routing (e.g., app1.example.com, app2.example.com).
- Perform path-based routing (e.g., /api, /admin).
- Enable SSL/TLS termination.
- Centralize routing and load balancing.
- Reduce cloud LoadBalancer costs.
### Installing NGINX Ingress Controller

```bash
# Apply the official manifest:
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml

# Check the controller:
kubectl get pods -n ingress-nginx

# Check the service:
kubectl get svc -n ingress-nginx
```
### Deploy an Application

```bash
# Create an NGINX deployment:
kubectl create deployment nginx --image=nginx
or
kubectl apply -f deployment.yaml

# Verify
kubectl get pods
```
### Create a Service

```bash
# Expose the deployment:
kubectl expose deployment nginx \
--port=80 \
--target-port=80 \
--type=ClusterIP
or
kubectl apply -f clusterip.yaml

# Verify
kubectl get svc
```

## Create an Ingress Resource
Example `ingress.yaml`:
```bash
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: myapp.local
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx
            port:
              number: 80
```

Apply it:
```bash
kubectl apply -f ingress.yaml

# Verify
kubectl get ingress
```

---
## Host-Based Routing
Routes traffic based on the hostname.
```bash
app1.example.com
        │
        ▼
   app1-service

app2.example.com
        │
        ▼
   app2-service
```

Example:
```bash
rules:
- host: app1.example.com
  http:
    paths:
    - path: /
      pathType: Prefix
      backend:
        service:
          name: app1-service
          port:
            number: 80
```

---
## Path-Based Routing
Routes traffic based on the URL path.
```bash
example.com/api
        │
        ▼
   api-service

example.com/web
        │
        ▼
   web-service
```

Example:
```bash
rules:
- host: example.com
  http:
    paths:
    - path: /api
      pathType: Prefix
      backend:
        service:
          name: api-service
          port:
            number: 80

    - path: /
      pathType: Prefix
      backend:
        service:
          name: frontend-service
          port:
            number: 80
```
---
## TLS/HTTPS Example
```bash
spec:
  tls:
  - hosts:
    - myapp.example.com
    secretName: tls-secret
```

Create the TLS secret:
```bash
kubectl create secret tls tls-secret \
--cert=tls.crt \
--key=tls.key
```

---
## Useful Commands

```bash
# Get all pods
kubectl get pods

# Check services
kubectl get svc

# Check Ingress
kubectl get ingress

# Describe the Ingress
kubectl describe ingress nginx-ingress

# Check Endpoints
kubectl get endpoints

# View Ingress Controller Pods:
kubectl get pods -n ingress-nginx

# View Controller Logs:
kubectl logs -n ingress-nginx deploy/ingress-nginx-controller

# Delete Ingress:
kubectl delete ingress nginx-ingress
```