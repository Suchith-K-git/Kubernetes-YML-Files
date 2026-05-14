# Kubernetes Resource Configurations

This repository contains YAML manifests for various Kubernetes workload resource types, covering Pods, Deployments, ReplicaSets, DaemonSets, and Jobs.

---

## 📁 File Overview

| File | Resource Type | Description |
|---|---|---|
| `pod.yml` | Pod | Basic Pod definition (1-hour lifespan) |
| `pod-1.yml` | Pod | Pod with namespace definition |
| `Deployment.yml` | Deployment | Manages rolling updates and desired state |
| `ReplicaSet.yml` | ReplicaSet | Ensures a stable set of replica Pods |
| `ReplicaSet-1.yml` | ReplicaSet | ReplicaSet using `In` operator in selector |
| `ReplicaSet-2.yml` | ReplicaSet | ReplicaSet using `Exists` operator in selector |
| `DaemonSet.yml` | DaemonSet | Runs a Pod on every (or selected) node |
| `Job.yml` | Job | Executes a task and completes |

---

## 🧱 Resource Descriptions

### Pod (`pod.yml`, `pod-1.yml`)

A **Pod** is the smallest deployable unit in Kubernetes. It wraps one or more containers that share the same network namespace and storage.

- `pod.yml` — Basic Pod created for a short-lived (~1 hour) workload.
- `pod-1.yml` — Same concept but scoped to a specific **namespace**, which allows logical isolation within a cluster.

**When to use:** Direct Pod definitions are rarely used in production. They are mostly used for quick debugging or testing.

```yaml
# Example structure
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

### Deployment (`Deployment.yml`)

A **Deployment** manages a set of identical Pods and ensures the desired number of replicas are always running. It supports rolling updates and rollbacks.

**When to use:** Use for stateless applications where you need scalability, rolling updates, and self-healing.

```yaml
# Example structure
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

### ReplicaSet (`ReplicaSet.yml`, `ReplicaSet-1.yml`, `ReplicaSet-2.yml`)

A **ReplicaSet** ensures that a specified number of Pod replicas are running at any given time. Deployments manage ReplicaSets under the hood — it's generally recommended to use Deployments rather than managing ReplicaSets directly.

**When to use:** Mostly managed automatically by Deployments. Use directly only when you need fine-grained control over Pod replicas without update strategies.

ReplicaSets support two types of selectors — `matchLabels` (simple equality) and `matchExpressions` (set-based). The latter supports operators like `In`, `NotIn`, `Exists`, and `DoesNotExist`.

---

#### `ReplicaSet-1.yml` — Using the `In` Operator

The `In` operator matches Pods whose label value for a given key is **one of a specified set of values**. This is useful when you want the ReplicaSet to manage Pods across multiple label variants.

```yaml
# Example structure
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

> The ReplicaSet will manage any Pod where the `environment` label is either `dev` or `staging`.

---

#### `ReplicaSet-2.yml` — Using the `Exists` Operator

The `Exists` operator matches Pods that **have a specific label key present**, regardless of its value. This is useful when you want to select Pods based on the presence of a label, not its specific value.

```yaml
# Example structure
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

> The ReplicaSet will manage any Pod that has the `environment` label set to **any** value.

---

#### Selector Operator Quick Reference

| Operator | Behavior |
|---|---|
| `In` | Key's value must match one of the listed values |
| `NotIn` | Key's value must not match any of the listed values |
| `Exists` | Key must be present (any value is accepted) |
| `DoesNotExist` | Key must not be present on the Pod |

---

### DaemonSet (`DaemonSet.yml`)

A **DaemonSet** ensures that all (or selected) nodes in the cluster run a copy of a specific Pod. When a new node is added to the cluster, a Pod is automatically scheduled on it.

**When to use:** Cluster-wide infrastructure tasks such as:
- Log collectors (Fluentd, Filebeat)
- Monitoring agents (Prometheus Node Exporter)
- Network plugins (Calico, Weave)

```yaml
# Example structure
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

### Job (`Job.yml`)

A **Job** creates one or more Pods and ensures they successfully complete their task. Once all Pods complete, the Job is marked as finished. Unlike Deployments, Jobs are not meant to run indefinitely.

**When to use:**
- Batch processing
- Database migrations
- One-time scripts or data processing tasks

```yaml
# Example structure
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

## 🔄 Resource Lifecycle Comparison

| Resource | Runs Until | Self-Healing | Rolling Updates | Node-Affinity |
|---|---|---|---|---|
| Pod | Manually deleted | ❌ | ❌ | Optional |
| Deployment | Always (desired state) | ✅ | ✅ | Optional |
| ReplicaSet | Always (desired state) | ✅ | ❌ | Optional |
| DaemonSet | Always (on every node) | ✅ | ✅ | Per node |
| Job | Task completion | ❌ (retry on failure) | ❌ | Optional |

---

## 🚀 Quick Start

### Prerequisites

- A running Kubernetes cluster (local: [minikube](https://minikube.sigs.k8s.io/) or [kind](https://kind.sigs.k8s.io/))
- `kubectl` configured and connected to your cluster

### Apply Resources

```bash
# Apply individual files
kubectl apply -f pod.yml
kubectl apply -f pod-1.yml
kubectl apply -f Deployment.yml
kubectl apply -f ReplicaSet.yml
kubectl apply -f DaemonSet.yml
kubectl apply -f Job.yml

# Or apply all at once
kubectl apply -f .
```

### Check Status

```bash
# List all resources
kubectl get all

# Watch Pod status in real time
kubectl get pods -w

# Describe a resource for details/events
kubectl describe pod <pod-name>
kubectl describe deployment <deployment-name>

# Check Job completion
kubectl get jobs
kubectl logs job/<job-name>
```

### Delete Resources

```bash
# Delete individual resources
kubectl delete -f Deployment.yml

# Delete all resources in directory
kubectl delete -f .
```

---

## 🗂️ Namespaces

`pod-1.yml` includes a namespace definition. To work with a specific namespace:

```bash
# Create a namespace
kubectl create namespace my-namespace

# List resources in a namespace
kubectl get all -n my-namespace

# Apply a manifest into a namespace
kubectl apply -f pod-1.yml -n my-namespace
```

---

## 📌 Best Practices

- Always set **resource requests and limits** on containers to prevent resource starvation.
- Use **labels and selectors** consistently across all manifests for easy grouping and querying.
- Prefer **Deployments** over bare ReplicaSets or Pods for production workloads.
- Use **namespaces** to isolate environments (dev, staging, prod) within the same cluster.
- For Jobs, always set `restartPolicy: Never` or `OnFailure` and configure `backoffLimit` appropriately.
- Use `kubectl diff -f <file>` before applying changes to preview what will change.

---

## 📝 Commit History

| File | Commit Message 
|---|---|---|
| `pod.yml` | Pod creation for 1 hr 
| `ReplicaSet.yml` | ReplicaSet.yml 
| `Deployment.yml` | Fix indentation in Deployment.yml 
| `ReplicaSet-1.yml` | Update ReplicaSet-1.yml 
| `ReplicaSet-2.yml` | Update ReplicaSet-2.yml 
| `pod-1.yml` | Add namespace definition to pod configuration
| `DaemonSet.yml` | DaemonSet.yml 
| `Job.yml` | Job.yml Execute the task and goes to completion state 

---

## 📚 References

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [kubectl Cheat Sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/kubernetes-api/)
