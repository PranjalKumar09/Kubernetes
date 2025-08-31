

# 🚀 **EKS + AWS Load Balancer Controller Setup Guide**

## 1. **EKS Cluster Setup**

### Install `eksctl`

```bash
curl --silent --location "https://github.com/weaveworks/eksctl/releases/latest/download/eksctl_$(uname -s)_amd64.tar.gz" \
  | tar xz -C /tmp
sudo mv /tmp/eksctl /usr/local/bin
eksctl version
```

### Create Cluster

```bash
eksctl create cluster \
  --name three-tier-cluster \
  --region us-west-2 \
  --node-type t2.medium \
  --nodes-min 2 \
  --nodes-max 2
```

### Configure `kubectl`

```bash
aws eks update-kubeconfig --region us-west-2 --name three-tier-cluster
kubectl get nodes
```

ℹ️ **Note**: An EKS cluster creates a CloudFormation stack, which is **regional**.

---

## 2. **AWS Load Balancer Controller Setup**

### Install IAM Policy

```bash
curl -O https://raw.githubusercontent.com/kubernetes-sigs/aws-load-balancer-controller/v2.5.4/docs/install/iam_policy.json
aws iam create-policy \
  --policy-name AWSLoadBalancerControllerIAMPolicy \
  --policy-document file://iam_policy.json
```

### Associate IAM OIDC Provider

```bash
eksctl utils associate-iam-oidc-provider \
  --region=us-west-2 \
  --cluster=three-tier-cluster \
  --approve
```

### Create Service Account + IAM Role

```bash
eksctl create iamserviceaccount \
  --cluster=three-tier-cluster \
  --namespace=kube-system \
  --name=aws-load-balancer-controller \
  --role-name AmazonEKSLoadBalancerControllerRole \
  --attach-policy-arn=arn:aws:iam::<account-id>:policy/AWSLoadBalancerControllerIAMPolicy \
  --approve \
  --region=us-west-2
```

### Install Controller with Helm

```bash
sudo snap install helm --classic
helm repo add eks https://aws.github.io/eks-charts
helm repo update eks

helm install aws-load-balancer-controller eks/aws-load-balancer-controller \
  -n kube-system \
  --set clusterName=three-tier-cluster \
  --set serviceAccount.create=false \
  --set serviceAccount.name=aws-load-balancer-controller \
  --set region=us-west-2
```

### Verify Installation

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl get pods -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller --watch
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
```

---

## 3. **Ingress with ALB**

Ingress acts as a **traffic director** to route external traffic (HTTP/HTTPS) into your services.

### Example Ingress

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: mainlb
  namespace: three-tier
  annotations:
    kubernetes.io/ingress.class: alb
    alb.ingress.kubernetes.io/scheme: internet-facing
    alb.ingress.kubernetes.io/target-type: ip
    alb.ingress.kubernetes.io/listen-ports: '[{"HTTP": 80}]'
spec:
  rules:
  - http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api
            port:
              number: 3500
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend
            port:
              number: 3000
```

✅ Pods running:

```
api         → port 3500
frontend    → port 3000
mongodb     → backend DB
```

---

## 4. **Troubleshooting AWS Load Balancer Controller**

### Common Failure Points

1. **IAM Role Issues** 🔐

   * **Symptom**: Pods stuck `Pending` or `CrashLoopBackOff`
   * **Fix**: Verify trust policy + role annotations

2. **Wrong Cluster Name** 📝

   * **Symptom**: Controller runs but ALBs aren’t created
   * **Fix**: Ensure `--set clusterName=<eks-cluster-name>` matches exactly

3. **Service Account Misconfig** ⚠️

   * **Symptom**: Logs show permission errors
   * **Fix**: Ensure annotation:

     ```yaml
     annotations:
       eks.amazonaws.com/role-arn: arn:aws:iam::<account>:role/AmazonEKSLoadBalancerControllerRole
     ```

### Debug Commands

```bash
kubectl get deployment -n kube-system aws-load-balancer-controller
kubectl logs -n kube-system -l app.kubernetes.io/name=aws-load-balancer-controller
aws iam get-role --role-name AmazonEKSLoadBalancerControllerRole
kubectl get serviceaccount aws-load-balancer-controller -n kube-system -o yaml
```

---

## 5. **Professional Enhancements**

### 1. Enable HTTPS / SSL

```yaml
annotations:
  alb.ingress.kubernetes.io/certificate-arn: arn:aws:acm:region:account:certificate/id
  alb.ingress.kubernetes.io/ssl-policy: ELBSecurityPolicy-TLS13-1-2-2021-06
  alb.ingress.kubernetes.io/listen-ports: '[{"HTTP":80},{"HTTPS":443}]'
```

### 2. Use Custom Domain

```yaml
annotations:
  external-dns.alpha.kubernetes.io/hostname: app.yourdomain.com
spec:
  rules:
  - host: app.yourdomain.com
```

### 3. Canary Deployments

```yaml
annotations:
  alb.ingress.kubernetes.io/actions.canary: |
    { "type": "forward",
      "forwardConfig": {
        "targetGroups": [
          {"serviceName":"frontend-v2","servicePort":3000,"weight":20},
          {"serviceName":"frontend","servicePort":3000,"weight":80}
        ]
      }
    }
```

### 4. Monitoring & Logs

```yaml
annotations:
  alb.ingress.kubernetes.io/load-balancer-attributes: >
    access_logs.s3.enabled=true,
    access_logs.s3.bucket=alb-logs-bucket,
    access_logs.s3.prefix=logs
```

### 5. Security Hardening

```yaml
annotations:
  alb.ingress.kubernetes.io/wafv2-acl-arn: arn:aws:wafv2:region:account:regional/webacl/id
  alb.ingress.kubernetes.io/response-headers-policy: arn:aws:elasticloadbalancing:region:account:response-headers-policy/id
```

---

## 6. **Professional Checklist**

✅ Use official AWS policies or Terraform modules
✅ Validate IAM policies before applying
✅ Use `eksctl` or IaC (Terraform/CloudFormation) for reproducibility
✅ Implement HTTPS + domain from day one
✅ Set up monitoring/logging immediately
✅ Harden security with WAF + headers
✅ Plan for rolling updates (zero downtime)
✅ Always verify existing resources before creation

---

## 7. **Key Learning Outcomes**

* Debugging **IAM + OIDC + RBAC**
* Understanding **Kubernetes Ingress + ALB integration**
* Real-world troubleshooting workflow
* Applying **IaC best practices**
* Securing and scaling production workloads

----
----
## ⚡ **Networking in EKS – Who Does What**

### 1. **ALB (Application Load Balancer)** 🌍

* **Purpose**: Expose your Kubernetes apps **to the outside world (internet)**
* **Created by**: AWS Load Balancer Controller (based on your `Ingress` manifest)
* **When to use**:

  * You need users to access your app from outside (browser, API clients, etc.)
  * For HTTPS termination (TLS/SSL)
  * For custom domains (via Route53/DNS)

👉 **Think of ALB as the *front door* to your cluster.**

---

### 2. **Service (ClusterIP / NodePort / LoadBalancer)** 🔄

* **Purpose**: Internal communication **between pods inside the cluster**
* **Types**:

  * `ClusterIP` → Default, only accessible inside the cluster (most common for microservice-to-microservice comms)
  * `NodePort` → Exposes on every node (used rarely, for debugging)
  * `LoadBalancer` → Creates an external LB (but with ALB controller, you usually rely on `Ingress` instead)

👉 **Think of a Service as the *stable internal address* for your pods.**

---

### 3. **Ingress** 🛂

* **Purpose**: Smart **router inside the cluster** that defines how external traffic is mapped to internal services
* **Works with**: ALB (created by controller)
* **When to use**:

  * Path-based routing (`/api` → API service, `/` → frontend)
  * Host-based routing (`api.example.com` → API, `app.example.com` → frontend)
  * Enforce HTTPS, redirects, canary, WAF policies

👉 **Think of Ingress as the *receptionist* telling ALB where to send traffic inside the cluster.**

---

## ✅ **Rule of Thumb**

* **External traffic (Internet → Cluster)** = **ALB + Ingress**
* **Internal traffic (Pod → Pod / Service → DB)** = **Service (ClusterIP)**

---

⚡ Example:

* **Outer World**:
  User → **ALB** → **Ingress** → **Frontend / API Services**
* **Inner Communication**:
  API Service → **ClusterIP Service** → MongoDB Pod




====


Extra knowledge
