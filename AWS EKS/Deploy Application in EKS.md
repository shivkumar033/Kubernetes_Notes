## 1. What is EKS?

**Amazon EKS (Elastic Kubernetes Service)** is AWS's managed Kubernetes service.

```shell
EKS = Kubernetes + AWS managed control plane
```

For this lab:
```shell
EKS
 ↓
Fargate
 ↓
Kubernetes Pods
 ↓
Service
 ↓
Ingress
 ↓
AWS Load Balancer Controller
 ↓
Application Load Balancer
 ↓
Internet
```

---
# 2. Prerequisites

Required tools:
```shell
aws --version
kubectl version --client
eksctl version
helm version
```

Configure AWS:
```shell
aws configure
```

Verify:
```shell
aws sts get-caller-identity
```

---
# 3. Create EKS Cluster

Set variables:
```shell
$REGION="us-east-1"
$CLUSTER="demo-cluster-1"
```

Create cluster with Fargate:
```shell
eksctl create cluster `
  --name demo-cluster-1 `
  --region us-east-1 `
  --fargate
```

Check:
```shell
eksctl get cluster
```

```shell
aws eks describe-cluster `
  --name demo-cluster-1 `
  --region us-east-1 `
  --query "cluster.status"
```

Expected:
```shell
ACTIVE
```

---
# 4. Configure kubectl

```shell
aws eks update-kubeconfig `
  --name demo-cluster-1 `
  --region us-east-1
```

Verify:
```shell
kubectl cluster-info
```

```shell
kubectl get pods -A
```

---
# 5. Create Application Namespace

```shell
kubectl create namespace game-2048
```

Check:
```shell
kubectl get namespace
```

---
# 6. Create Fargate Profile

```shell
eksctl create fargateprofile `
  --cluster demo-cluster-1 `
  --region us-east-1 `
  --name game-2048 `
  --namespace game-2048
```

Check:
```shell
eksctl get fargateprofile `
  --cluster demo-cluster-1 `
  --region us-east-1
```

Purpose:
```shell
game-2048 namespace
        ↓
Fargate Profile
        ↓
Fargate
        ↓
2048 Pods
```

---
# 7. Deploy 2048 Application

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/examples/2048/2048_full.yaml
```

Check:
```shell
kubectl get all -n game-2048
```

Pods:
```shell
kubectl get pods -n game-2048
```

Deployment:
```shell
kubectl get deployment -n game-2048
```

Service:
```shell
kubectl get svc -n game-2048
```

---
# 8. Configure OIDC

OIDC allows Kubernetes ServiceAccounts to assume AWS IAM roles.
```shell
ServiceAccount
      ↓
     OIDC
      ↓
   IAM Role
      ↓
 IAM Policy
      ↓
  AWS APIs
```

Command:
```shell
eksctl utils associate-iam-oidc-provider `
  --cluster demo-cluster-1 `
  --region us-east-1 `
  --approve
```

---

# 9. Get AWS Account ID

```shell
$ACCOUNT_ID = aws sts get-caller-identity `
  --query Account `
  --output text
```

Check:
```shell
echo $ACCOUNT_ID
```

---

# 10. Get VPC ID

```shell
$VPC_ID = aws eks describe-cluster `
  --name demo-cluster-1 `
  --region us-east-1 `
  --query "cluster.resourcesVpcConfig.vpcId" `
  --output text
```

Check:
```
echo $VPC_ID
```

---

# 11. Download ALB Controller IAM Policy

```shell
Invoke-WebRequest `
  -Uri "https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/install/iam_policy.json" `
  -OutFile "iam_policy.json"
```

---

# 12. Create IAM Policy

```shell
aws iam create-policy `
  --policy-name AWSLoadBalancerControllerIAMPolicy `
  --policy-document file://iam_policy.json
```

Policy ARN:
```shell
arn:aws:iam::<ACCOUNT_ID>:policy/AWSLoadBalancerControllerIAMPolicy
```

---

# 13. Create IAM Service Account

```shell
eksctl create iamserviceaccount `
  --cluster demo-cluster-1 `
  --namespace kube-system `
  --name aws-load-balancer-controller `
  --attach-policy-arn arn:aws:iam::${ACCOUNT_ID}:policy/AWSLoadBalancerControllerIAMPolicy `
  --override-existing-serviceaccounts `
  --region us-east-1 `
  --approve
```

Verify:
```shell
kubectl get serviceaccount `
  aws-load-balancer-controller `
  -n kube-system
```

---

# 14. Install AWS Load Balancer Controller

Add Helm repository:
```shell
helm repo add eks https://aws.github.io/eks-charts
```

Update:
```shell
helm repo update eks
```

Install:
```shell
helm install aws-load-balancer-controller `
  eks/aws-load-balancer-controller `
  -n kube-system `
  --set clusterName=demo-cluster-1 `
  --set serviceAccount.create=false `
  --set serviceAccount.name=aws-load-balancer-controller `
  --set region=us-east-1 `
  --set vpcId=$VPC_ID `
  --version 1.14.0
```

---

# 15. Verify Controller

```shell
kubectl get deployment `
  aws-load-balancer-controller `
  -n kube-system
```

Check Pods:
```
kubectl get pods -n kube-system
```

Expected:
```shell
aws-load-balancer-controller   2/2   Running
```

---

# 16. Check Ingress

```shell
kubectl get ingress -n game-2048
```

After the ALB is created:
```shell
ADDRESS
k8s-game2048-xxxxx.us-east-1.elb.amazonaws.com
```

Get only the DNS:
```shell
kubectl get ingress ingress-2048 `
  -n game-2048 `
  -o jsonpath="{.status.loadBalancer.ingress[0].hostname}"
```

Open the returned address in your browser.

🎮 **2048 Game should appear.**

# 17. Troubleshooting Commands

### Pods
```shell
kubectl get pods -n game-2048
```

### Pod details
```shell
kubectl describe pod -n game-2048 <POD_NAME>
```

### Application logs
```shell
kubectl logs -n game-2048 <POD_NAME>
```

### Ingress details
```shell
kubectl describe ingress -n game-2048
```

### Controller logs
```
kubectl logs `
  -n kube-system `
  -l app.kubernetes.io/instance=aws-load-balancer-controller
```

### Controller status
```shell
kubectl get deployment `
  aws-load-balancer-controller `
  -n kube-system
```

---
# 18. Delete Everything

Delete application:
```shell
kubectl delete -f https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.14.1/docs/examples/2048/2048_full.yaml
```

Delete cluster:
```shell
eksctl delete cluster `
  --name demo-cluster-1 `
  --region us-east-1
```