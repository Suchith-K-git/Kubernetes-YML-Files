# Kubernetes Workload Resource Examples

This repository contains YAML manifests for various Kubernetes workload resource types, covering Pods, Deployments, ReplicaSets, DaemonSets, and Jobs.

---

# 📁 File Overview

| File | Resource Type | Description |
|---|---|---|
| `pod.yml` | Pod | Basic Pod definition for a temporary workload |
| `pod-1.yml` | Pod | Pod with namespace definition |
| `Deployment.yml` | Deployment | Manages rolling updates and desired state |
| `ReplicaSet.yml` | ReplicaSet | Ensures a stable set of replica Pods |
| `ReplicaSet-1.yml` | ReplicaSet | ReplicaSet using `In` operator in selector |
| `ReplicaSet-2.yml` | ReplicaSet | ReplicaSet using `Exists` operator in selector |
| `DaemonSet.yml` | DaemonSet | Runs a Pod on every (or selected) node |
| `Job.yml` | Job | Executes a task and completes successfully |

---

# 📂 Repository Structure

```text
.
├── pod.yml
├── pod-1.yml
├── Deployment.yml
├── ReplicaSet.yml
├── ReplicaSet-1.yml
├── ReplicaSet-2.yml
├── DaemonSet.yml
├── Job.yml
└── README.md
```

---

# 🧱 Resource Descriptions

## Pod (`pod.yml`, `pod-1.yml`)

A **Pod** is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace and storage.

- `pod.yml` — Example Pod running a temporary workload.
- `pod-1.yml` — Pod scoped to a specific namespace for logical isolation.

### Common Restart Policies

- `Always` (default)
- `OnFailure`
- `Never`

### When to use

Direct Pod definitions are rarely used in production. They are mainly used for:
- Debugging
- Testing
- Learning Kubernetes basics

### Example Structure

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
  namespace: my-namespace

spec:
  containers:
    - name: my-container
      image: nginx:latest
```

---

## Deployment (`Deployment.yml`)

A **Deployment** manages a set of identical Pods and ensures the desired number of replicas are always running. It supports rolling updates and rollbacks.

### Features

- Self-healing
- Rolling updates
- Rollbacks
- Scaling
- Desired state management

### When to use

Use Deployments for:
- Stateless applications
- Web applications
- APIs
- Microservices

### Example Structure

```yaml
apiVersion: apps/v1
kind: Deployment

metadata:
  name: my-deployment

spec:
  replicas: 3

  selector:
    matchLabels:
      app: my-app

  template:
    metadata:
      labels:
        app: my-app

    spec:
      containers:
        - name: my-container
          image: nginx:latest
```

---

## ReplicaSet (`ReplicaSet.yml`, `ReplicaSet-1.yml`, `ReplicaSet-2.yml`)

A **ReplicaSet** ensures that a specified number of Pod replicas are running at any given time.

Deployments manage ReplicaSets internally, so ReplicaSets are rarely used directly in production.

### Features

- Maintains desired replica count
- Self-healing
- Supports label selectors

### When to use

Use ReplicaSets directly only when:
- You need low-level replica management
- You do not require rolling updates

---

## `ReplicaSet-1.yml` — Using the `In` Operator

The `In` operator matches Pods whose label value belongs to a defined list.

### Example Structure

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: replicaset-in

spec:
  replicas: 3

  selector:
    matchExpressions:
      - key: environment
        operator: In
        values:
          - dev
          - staging

  template:
    metadata:
      labels:
        environment: dev

    spec:
      containers:
        - name: my-container
          image: nginx:latest
```

### Explanation

The ReplicaSet manages Pods where:

```text
environment = dev OR staging
```

---

## `ReplicaSet-2.yml` — Using the `Exists` Operator

The `Exists` operator matches Pods that contain a specific label key regardless of its value.

### Example Structure

```yaml
apiVersion: apps/v1
kind: ReplicaSet

metadata:
  name: replicaset-exists

spec:
  replicas: 3

  selector:
    matchExpressions:
      - key: environment
        operator: Exists

  template:
    metadata:
      labels:
        environment: production

    spec:
      containers:
        - name: my-container
          image: nginx:latest
```

### Explanation

The ReplicaSet manages any Pod that contains:

```text
environment=<any-value>
```

---

## 🔎 Selector Operator Quick Reference

| Operator | Behavior |
|---|---|
| `In` | Key value must match one of the listed values |
| `NotIn` | Key value must not match listed values |
| `Exists` | Key must exist |
| `DoesNotExist` | Key must not exist |

---

## DaemonSet (`DaemonSet.yml`)

A **DaemonSet** ensures that all (or selected) nodes run a copy of a Pod.

When a new node joins the cluster, Kubernetes automatically schedules the DaemonSet Pod onto that node.

### Common Use Cases

- Log collection
- Monitoring agents
- Node exporters
- Security agents
- Networking plugins

### Examples

- Fluentd
- Filebeat
- Prometheus Node Exporter
- Calico

### Example Structure

```yaml
apiVersion: apps/v1
kind: DaemonSet

metadata:
  name: my-daemonset

spec:
  selector:
    matchLabels:
      app: my-daemon

  template:
    metadata:
      labels:
        app: my-daemon

    spec:
      containers:
        - name: my-container
          image: fluentd:latest
```

---

## Job (`Job.yml`)

A **Job** creates one or more Pods and ensures they successfully complete their task.

Unlike Deployments, Jobs are not designed to run forever.

### Features

- Runs until task completion
- Retries failed Pods
- Supports parallel execution

### When to use

Jobs are useful for:
- Batch processing
- One-time scripts
- Database migrations
- Scheduled tasks
- Data processing

### Example Structure

```yaml
apiVersion: batch/v1
kind: Job

metadata:
  name: my-job

spec:
  template:
    spec:
      containers:
        - name: my-job-container
          image: busybox
          command: ["sh", "-c", "echo Hello Kubernetes && sleep 10"]

      restartPolicy: Never

  backoffLimit: 4
```

---

## StatefulSet (Concept Overview)

A **StatefulSet** is used for stateful applications where Pods require:

- Stable network identities
- Persistent storage
- Ordered deployment and scaling

### Common Examples

- MySQL
- MongoDB
- Kafka
- Cassandra

---

## Service (Concept Overview)

A **Service** exposes a set of Pods using a stable network endpoint.

### Service Types

| Type | Purpose |
|---|---|
| `ClusterIP` | Internal cluster communication |
| `NodePort` | External access via node port |
| `LoadBalancer` | External load balancer integration |

### Use Cases

- Internal communication
- Load balancing
- External application access

---

# 🔄 Resource Lifecycle Comparison

| Resource | Runs Until | Self-Healing | Rolling Updates | Node Scope |
|---|---|---|---|---|
| Pod | Manually deleted | ❌ | ❌ | Optional |
| Deployment | Desired state maintained | ✅ | ✅ | Optional |
| ReplicaSet | Desired replica count maintained | ✅ | ❌ | Optional |
| DaemonSet | One Pod per node | ✅ | ✅ | Per node |
| Job | Task completion | Retry on failure | ❌ | Optional |

---

# 🚀 Quick Start

## Prerequisites

Install:

- Kubernetes cluster
  - Minikube
  - Kind
  - EKS
  - AKS
  - GKE
- `kubectl`

---

## Apply Resources

### Apply Individual Files

```bash
kubectl apply -f pod.yml
kubectl apply -f pod-1.yml
kubectl apply -f Deployment.yml
kubectl apply -f ReplicaSet.yml
kubectl apply -f ReplicaSet-1.yml
kubectl apply -f ReplicaSet-2.yml
kubectl apply -f DaemonSet.yml
kubectl apply -f Job.yml
```

### Apply All Files

```bash
kubectl apply -f .
```

---

# 📊 Verify Resources

## List All Resources

```bash
kubectl get all
```

## Watch Pod Status

```bash
kubectl get pods -w
```

## Describe Resources

```bash
kubectl describe pod <pod-name>

kubectl describe deployment <deployment-name>

kubectl describe replicaset <replicaset-name>

kubectl describe daemonset <daemonset-name>

kubectl describe job <job-name>
```

---

# 📜 Logs and Debugging

## View Logs

```bash
kubectl logs <pod-name>
```

## Execute Inside a Pod

```bash
kubectl exec -it <pod-name> -- /bin/sh
```

## Port Forward

```bash
kubectl port-forward pod/<pod-name> 8080:80
```

---

# 🗂️ Namespaces

`pod-1.yml` includes a namespace definition.

## Create Namespace

```bash
kubectl create namespace my-namespace
```

## List Resources in Namespace

```bash
kubectl get all -n my-namespace
```

## Apply Manifest in Namespace

```bash
kubectl apply -f pod-1.yml -n my-namespace
```

---

# 🛠️ Useful Commands

## Delete Resources

### Delete Individual Resource

```bash
kubectl delete -f Deployment.yml
```

### Delete All Resources

```bash
kubectl delete -f .
```

---

# 📌 Best Practices

- Always define resource requests and limits
- Avoid using the `latest` image tag in production
- Use labels and selectors consistently
- Prefer Deployments over Pods and ReplicaSets
- Use namespaces for environment isolation
- Store sensitive data using Secrets
- Use ConfigMaps for configuration management
- Configure readiness and liveness probes
- Use `kubectl diff` before applying changes
- Set proper restart policies for Jobs

---

# 📝 Commit History

| File | Commit Message |
|---|---|
| `pod.yml` | Pod creation for temporary workload |
| `ReplicaSet.yml` | ReplicaSet.yml |
| `Deployment.yml` | Fix indentation in Deployment.yml |
| `ReplicaSet-1.yml` | Update ReplicaSet-1.yml |
| `ReplicaSet-2.yml` | Update ReplicaSet-2.yml |
| `pod-1.yml` | Add namespace definition to pod configuration |
| `DaemonSet.yml` | DaemonSet.yml |
| `Job.yml` | Execute task and move to completion state |

---

# 📚 References

- Kubernetes Official Documentation  
  https://kubernetes.io/docs/

- kubectl Cheat Sheet  
  https://kubernetes.io/docs/reference/kubectl/cheatsheet/

- Kubernetes API Reference  
  https://kubernetes.io/docs/reference/kubernetes-api/

---

# ✅ Summary

This repository demonstrates the core Kubernetes workload resources and their usage patterns.

It is useful for:

- Kubernetes beginners
- DevOps practice
- Interview preparation
- YAML manifest learning
- Kubernetes hands-on labs
- GitHub portfolio projects
