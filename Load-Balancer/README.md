# AWS LoadBalancer Setup for kubeadm Cluster (1 Master + 1 Worker)

# Architecture

```text
Internet
   ↓
AWS Load Balancer (ELB/NLB)
   ↓
Kubernetes Service (LoadBalancer)
   ↓
Pods
```

This setup uses:

* Kubernetes (kubeadm)
* AWS EC2
* AWS Cloud Controller Manager

---

# STEP 1 — AWS EC2 Requirements

You need:

| Node   | Example Name |
| ------ | ------------ |
| Master | master-node  |
| Worker | worker-node  |

Both instances must:

* Be in SAME VPC
* Same subnet or routable subnet
* Same security group (recommended)
* Public IPv4 enabled

---

# STEP 2 — Security Group Configuration

Go to:

```text
AWS Console → EC2 → Security Groups
```

Add these inbound rules to BOTH instances:

| Type        | Port        | Source              |
| ----------- | ----------- | ------------------- |
| SSH         | 22          | Your IP             |
| HTTP        | 80          | 0.0.0.0/0           |
| HTTPS       | 443         | 0.0.0.0/0           |
| Custom TCP  | 30000-32767 | 0.0.0.0/0           |
| All Traffic | All         | Same Security Group |

IMPORTANT:

"All Traffic" between nodes is mandatory.

---

# STEP 3 — Create IAM Role

Go to:

```text
AWS Console → IAM → Roles → Create Role
```

Trusted Entity:

```text
EC2
```

Attach Policy:

```text
AdministratorAccess
```

Role Name:

```text
k8s-cloud-role
```

NOTE:

AdministratorAccess is for learning/testing only.

---

# STEP 4 — Attach IAM Role to BOTH EC2 Instances

Go to:

```text
AWS Console → EC2 → Instances
```

For BOTH nodes:

```text
Actions → Security → Modify IAM Role
```

Attach:

```text
k8s-cloud-role
```

---

# STEP 5 — Add Kubernetes Cluster Tag

Go to BOTH EC2 instances:

```text
Tags → Manage Tags
```

Add:

| Key                                | Value |
| ---------------------------------- | ----- |
| kubernetes.io/cluster/demo-cluster | owned |

IMPORTANT:

The SAME tag must exist on BOTH instances.

---

# STEP 6 — Verify Kubernetes Cluster

Run on MASTER node:

```bash
kubectl get nodes
```

Expected Output:

```text
NAME           STATUS   ROLES           AGE
master-node    Ready    control-plane
worker-node    Ready    <none>
```

---

# STEP 7 — Create Service Account

Run ONLY on MASTER node:

```bash
kubectl create serviceaccount aws-cloud-controller-manager -n kube-system
```

Create ClusterRoleBinding:

```bash
kubectl create clusterrolebinding aws-cloud-controller-manager \
  --clusterrole=cluster-admin \
  --serviceaccount=kube-system:aws-cloud-controller-manager
```

---

# STEP 8 — Configure kube-apiserver

Run on MASTER node:

```bash
sudo nano /etc/kubernetes/manifests/kube-apiserver.yaml
```

Find:

```yaml
- kube-apiserver
```

Add:

```yaml
- --cloud-provider=external
```

Example:

```yaml
spec:
  containers:
  - command:
    - kube-apiserver
    - --cloud-provider=external
```

Save file.

Wait 30 seconds.

---

# STEP 9 — Configure kube-controller-manager

Run:

```bash
sudo nano /etc/kubernetes/manifests/kube-controller-manager.yaml
```

Find:

```yaml
- kube-controller-manager
```

Add:

```yaml
- --cloud-provider=external
```

Example:

```yaml
- kube-controller-manager
- --cloud-provider=external
```

Save file.

Wait 30 seconds.

---

# STEP 10 — Create AWS Cloud Controller Manifest

Create file:

```bash
nano aws-cloud-controller-manager.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: aws-cloud-controller-manager
  namespace: kube-system

spec:
  selector:
    matchLabels:
      k8s-app: aws-cloud-controller-manager

  template:
    metadata:
      labels:
        k8s-app: aws-cloud-controller-manager

    spec:
      serviceAccountName: aws-cloud-controller-manager
      hostNetwork: true

      tolerations:
      - operator: Exists

      containers:
      - name: aws-cloud-controller-manager

        image: registry.k8s.io/provider-aws/cloud-controller-manager:v1.29.0

        command:
        - /bin/aws-cloud-controller-manager
        - --cloud-provider=aws
        - --configure-cloud-routes=false
        - --cluster-name=demo-cluster
        - --v=2
```

IMPORTANT:

This line is mandatory:

```yaml
- --configure-cloud-routes=false
```

Why?

Because Calico already manages Kubernetes pod networking.

Without this flag:

* AWS Cloud Controller Manager tries to manage pod routes
* Calico also manages routes
* Both conflict
* Controller crashes

Typical error:

```text
error running controllers: invalid CIDR[0]: <nil> (invalid CIDR address: )
```

Apply manifest:

```bash
kubectl apply -f aws-cloud-controller-manager.yaml
```

---

# STEP 11 — Verify Cloud Controller

Run:

```bash
kubectl get pods -n kube-system
```

Expected:

```text
aws-cloud-controller-manager   Running
```

Check logs:

```bash
kubectl logs -n kube-system -l k8s-app=aws-cloud-controller-manager
```

---

# STEP 12 — Deploy NGINX Application

Create file:

```bash
nano nginx-deployment.yaml
```

Paste:

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: nginx-deployment

spec:
  replicas: 2

  selector:
    matchLabels:
      app: nginx

  template:
    metadata:
      labels:
        app: nginx

    spec:
      containers:
      - name: nginx
        image: nginx

        ports:
        - containerPort: 80
```

Apply:

```bash
kubectl apply -f nginx-deployment.yaml
```

Verify:

```bash
kubectl get pods -o wide
```

---

# STEP 13 — Create AWS LoadBalancer Service

Create file:

```bash
nano nginx-loadbalancer.yaml
```

Paste:

```yaml
apiVersion: v1
kind: Service

metadata:
  name: nginx-loadbalancer

spec:
  type: LoadBalancer

  selector:
    app: nginx

  ports:
  - port: 80
    targetPort: 80
```

Apply:

```bash
kubectl apply -f nginx-loadbalancer.yaml
```

---

# STEP 14 — Verify Load Balancer Creation

Run:

```bash
kubectl get svc
```

Initially:

```text
EXTERNAL-IP   <pending>
```

Wait 2–5 minutes.

Then you should see:

```text
EXTERNAL-IP   a1b2c3d4e5.us-east-1.elb.amazonaws.com
```

AWS automatically creates:

* ELB/NLB
* Target Groups
* Health Checks
* Node Registration

---

# STEP 15 — Access Application

Open browser:

```text
http://<EXTERNAL-IP>
```

Example:

```text
http://a1b2c3d4e5.us-east-1.elb.amazonaws.com
```

Expected Output:

```text
Welcome to nginx!
```

---

# STEP 16 — Verify in AWS Console

Go to:

```text
AWS Console → EC2 → Load Balancers
```

You should see:

* AWS Load Balancer
* Registered Targets
* Health Checks
* Traffic Flow

---

# Traffic Flow

```text
Browser Request
      ↓
AWS Load Balancer
      ↓
Master / Worker Node
      ↓
Kubernetes Service
      ↓
NGINX Pods
```

---

# Useful Commands

## Check Nodes

```bash
kubectl get nodes
```

## Check Pods

```bash
kubectl get pods -o wide
```

## Check Services

```bash
kubectl get svc
```

## Describe Service

```bash
kubectl describe svc nginx-loadbalancer
```

## Check Controller Logs

```bash
kubectl logs -n kube-system -l k8s-app=aws-cloud-controller-manager
```

---

# Common Problems

## EXTERNAL-IP Pending

Possible causes:

* IAM role missing
* Cluster tag missing
* Controller not running
* Wrong cloud-provider configuration
* Security group issue

## AWS Cloud Controller CrashLoopBackOff

Possible causes:

* Missing:

```yaml
--configure-cloud-routes=false
```

* Cluster tag mismatch
* IAM role missing
* Incorrect cloud-provider setting

---

# Cleanup

## Delete Service

```bash
kubectl delete svc nginx-loadbalancer
```

## Delete Deployment

```bash
kubectl delete deployment nginx-deployment
```

## Delete Controller

```bash
kubectl delete -f aws-cloud-controller-manager.yaml
```

---

# Production Recommendation

For production Kubernetes on AWS, preferred setup is:

* AWS Load Balancer Controller
* NGINX Ingress Controller

Advantages:

* ALB Support
* HTTPS/SSL
* Domain Routing
* Path Routing
* Autoscaling
* WAF Integration

---

# Final Checklist

| Item                      | Status |
| ------------------------- | ------ |
| Nodes Ready               | ✅      |
| IAM Role Attached         | ✅      |
| Cluster Tag Added         | ✅      |
| Cloud Provider Configured | ✅      |
| Cloud Controller Running  | ✅      |
| NGINX Pods Running        | ✅      |
| LoadBalancer Created      | ✅      |
| EXTERNAL-IP Assigned      | ✅      |
| Application Accessible    | ✅      |
