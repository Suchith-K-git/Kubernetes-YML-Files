# Kubernetes ResourceQuota, Resource Requests, Limits, and LimitRange Practical

This lab demonstrates how a Kubernetes Namespace can enforce aggregate
resource quotas while individual containers define their own CPU and
memory requests and limits.

## 1. Project structure

Create a directory and place the following files inside it:

``` text
resource-quota-practical/
├── 01-namespace.yml
├── 02-resource-quota.yml
├── 03-limit-range.yml
├── 04-nginx-deployment.yml
├── 05-mysql-deployment.yml
├── 06-redis-deployment.yml
├── 07-backend-deployment.yml
└── README.md
```

## 2. Prerequisites

-   A working Kubernetes cluster.
-   `kubectl` installed and configured to access the cluster.
-   Permission to create Namespaces, ResourceQuotas, LimitRanges, and
    Deployments.
-   Sufficient CPU and memory capacity on the worker nodes.
-   Metrics Server installed if you want to use `kubectl top`.

Check cluster access:

``` bash
kubectl cluster-info
kubectl get nodes
```

## 3. YAML files

### 3.1 Create the Namespace --- `01-namespace.yml`

``` yaml
apiVersion: v1
kind: Namespace
metadata:
  name: development
```

The Namespace provides the scope for the quota, LimitRange, and
application workloads.

### 3.2 Create the ResourceQuota --- `02-resource-quota.yml`

``` yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: dev-quota
  namespace: development
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    pods: "10"
    services: "10"
    configmaps: "10"
    secrets: "10"
    persistentvolumeclaims: "5"
```

The quota applies to the entire `development` Namespace. It does not
divide resources equally among Pods and does not reserve node capacity.

  Quota                      Namespace-wide maximum
  ------------------------ ------------------------
  Total CPU requests                          4 CPU
  Total memory requests                         8Gi
  Total CPU limits                            8 CPU
  Total memory limits                          16Gi
  Pods                                           10
  Services                                       10
  ConfigMaps                                     10
  Secrets                                        10
  PersistentVolumeClaims                          5

### 3.3 Create the LimitRange --- `03-limit-range.yml`

``` yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: dev-limit-range
  namespace: development
spec:
  limits:
    - type: Container
      default:
        cpu: 500m
        memory: 512Mi
      defaultRequest:
        cpu: 100m
        memory: 128Mi
      min:
        cpu: 50m
        memory: 64Mi
      max:
        cpu: "2"
        memory: 2Gi
```

This LimitRange applies to each container in the Namespace:

-   `default`: default container limits when limits are omitted.
-   `defaultRequest`: default container requests when requests are
    omitted.
-   `min`: minimum permitted request/limit values.
-   `max`: maximum permitted request/limit values.

The explicit resource settings in the Deployments below take precedence
over the defaults. The minimum and maximum constraints still apply.

### 3.4 Nginx Deployment --- `04-nginx-deployment.yml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: development
spec:
  replicas: 4
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
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 200m
              memory: 256Mi
```

Four Pods are created. Each container requests 100m CPU and 128Mi
memory, with limits of 200m CPU and 256Mi memory.

### 3.5 MySQL Deployment --- `05-mysql-deployment.yml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql-deployment
  namespace: development
spec:
  replicas: 2
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
        - name: mysql
          image: mysql:8.0
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "DevPassword123!"
          ports:
            - containerPort: 3306
          resources:
            requests:
              cpu: 500m
              memory: 512Mi
            limits:
              cpu: 1000m
              memory: 1Gi
```

**Lab warning:** The password is hardcoded only for this demonstration.
Use a Kubernetes Secret for credentials in real environments. This
simple Deployment is not a production-ready database setup; production
MySQL needs appropriate persistent storage, backup, security, and
database architecture.

### 3.6 Redis Deployment --- `06-redis-deployment.yml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: redis-deployment
  namespace: development
spec:
  replicas: 2
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:7
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 200m
              memory: 256Mi
            limits:
              cpu: 500m
              memory: 512Mi
```

### 3.7 Backend placeholder Deployment --- `07-backend-deployment.yml`

``` yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
  namespace: development
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: nginx:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: 300m
              memory: 512Mi
            limits:
              cpu: 600m
              memory: 1Gi
```

Nginx is used as a placeholder image for the backend resource exercise.
The Nginx image listens on port 80 by default, so the container port is
set to 80. Replace this image and port with your actual backend
application when using one.

## 4. Apply the YAML files

Run these commands from the directory containing the YAML files. Apply
them in this order:

``` bash
kubectl apply -f 01-namespace.yml
kubectl apply -f 02-resource-quota.yml
kubectl apply -f 03-limit-range.yml

kubectl apply -f 04-nginx-deployment.yml
kubectl apply -f 05-mysql-deployment.yml
kubectl apply -f 06-redis-deployment.yml
kubectl apply -f 07-backend-deployment.yml
```

You can also apply all the YAML files in the current directory:

``` bash
kubectl apply -f .
```

The numbered filenames help keep the intended order clear. The Namespace
must exist before the namespaced objects are created.

## 5. Verify the resources

### Check the Namespace

``` bash
kubectl get namespaces
kubectl get namespace development
```

### Check the ResourceQuota

``` bash
kubectl get resourcequota -n development
kubectl describe resourcequota dev-quota -n development
```

In the detailed output, inspect `Used` and `Hard` for CPU requests,
memory requests, CPU limits, memory limits, and Pod count.

### Check the LimitRange

``` bash
kubectl get limitrange -n development
kubectl describe limitrange dev-limit-range -n development
```

### Check Deployments and Pods

``` bash
kubectl get deployments -n development
kubectl get pods -n development
kubectl get pods -n development -o wide
```

### Inspect a Pod's configured resources

``` bash
kubectl describe pod <pod-name> -n development
```

Or inspect the resource settings in YAML:

``` bash
kubectl get pod <pod-name> -n development -o yaml
```

### Check actual CPU and memory usage

``` bash
kubectl top pods -n development
kubectl top nodes
```

These `top` commands require Metrics Server. Requests and limits are
configured values; `kubectl top` reports measured usage.

## 6. Expected resource calculation

The four Deployments create 10 Pods in total.

  ------------------------------------------------------------------------------
  Application           Pods    Total CPU Total memory    Total CPU Total memory
                                 requests     requests       limits       limits
  ------------- ------------ ------------ ------------ ------------ ------------
  Nginx                    4         400m        512Mi         800m       1024Mi

  MySQL                    2        1000m       1024Mi        2000m       2048Mi

  Redis                    2         400m        512Mi        1000m       1024Mi

  Backend                  2         600m       1024Mi        1200m       2048Mi

  **Total**           **10** **2400m (2.4     **3072Mi   **5000m (5     **6144Mi
                                   CPU)**      (3Gi)**       CPU)**      (6Gi)**
  ------------------------------------------------------------------------------

These totals are within the configured ResourceQuota:

-   CPU requests: 2.4 of 4 CPU.
-   Memory requests: 3Gi of 8Gi.
-   CPU limits: 5 of 8 CPU.
-   Memory limits: 6Gi of 16Gi.
-   Pods: 10 of 10.

The quota is an aggregate namespace budget. It does not automatically
distribute resources among Pods or reserve those resources on worker
nodes. The developer specifies each container's requests and limits; the
scheduler places Pods on nodes with sufficient allocatable capacity for
their requests.

## 7. Practical experiments

Perform these experiments after the initial lab is running.

### Experiment 1: Try to create an additional Pod

The Namespace already has 10 Pods, which is its configured maximum.

``` bash
kubectl run extra-nginx \
  --image=nginx \
  --namespace=development
```

The Pod creation should be rejected because the Pod quota is exhausted.

Check the quota:

``` bash
kubectl describe resourcequota dev-quota -n development
```

### Experiment 2: Test the LimitRange maximum

Try to create a Pod whose container requests more than the LimitRange
maximum:

``` bash
kubectl run too-large \
  --image=nginx \
  --namespace=development \
  --dry-run=server \
  -o yaml \
  --overrides='{"spec":{"containers":[{"name":"nginx","image":"nginx","resources":{"requests":{"cpu":"3","memory":"128Mi"},"limits":{"cpu":"3","memory":"128Mi"}}}]}}'
```

The API server should reject it because 3 CPU exceeds the per-container
maximum of 2 CPU. This is a server-side dry run; it does not create a
Pod.

### Experiment 3: Observe LimitRange defaults

Create a small test Pod without specifying resources:

``` bash
kubectl run default-test \
  --image=nginx \
  --namespace=development \
  --dry-run=server \
  -o yaml
```

Inspect the generated Pod specification for default resource requests
and limits injected by the LimitRange. Because the namespace's Pod quota
is already full in the main lab, use a separate test Namespace or
temporarily adjust the quota if you want to create this Pod for
inspection.

### Experiment 4: Scale a Deployment

For example, try scaling Nginx:

``` bash
kubectl scale deployment nginx-deployment \
  --replicas=5 \
  -n development
```

The Deployment controller will attempt to create another Pod, but the
Namespace already uses all 10 Pod quota slots. The extra Pod cannot be
admitted until quota is available.

Restore the original replica count if needed:

``` bash
kubectl scale deployment nginx-deployment \
  --replicas=4 \
  -n development
```

## 8. Troubleshooting

### Check why a Pod was not created

``` bash
kubectl get pods -n development
kubectl describe pod <pod-name> -n development
kubectl get events -n development --sort-by=.metadata.creationTimestamp
```

Look for messages such as `exceeded quota` or LimitRange validation
errors.

### Check whether the quota is exhausted

``` bash
kubectl describe quota dev-quota -n development
```

Compare the `Used` and `Hard` values.

### Check whether the node has sufficient capacity

``` bash
kubectl get nodes
kubectl describe nodes
kubectl describe pod <pending-pod-name> -n development
```

A Pod can be within Namespace quota and still remain Pending if no
worker node has enough allocatable resources for its requests.

## 9. Important concepts

  -----------------------------------------------------------------------
  Kubernetes concept                  Responsibility
  ----------------------------------- -----------------------------------
  Namespace                           Logical separation and scope for
                                      namespaced resources

  ResourceQuota                       Enforces aggregate resource and
                                      object-count quotas in a Namespace

  LimitRange                          Defines per-container or per-Pod
                                      defaults and resource boundaries

  Requests                            Resource requirements used by the
                                      scheduler

  Limits                              Maximum CPU and memory consumption
                                      configured for a container

  Scheduler                           Selects a suitable worker node for
                                      a Pod
  -----------------------------------------------------------------------

## 10. Cleanup

Delete the Namespace to remove the lab resources created inside it:

``` bash
kubectl delete namespace development
```

Verify deletion:

``` bash
kubectl get namespaces
```

**Warning:** Deleting the Namespace also deletes its Deployments, Pods,
ResourceQuota, LimitRange, and other namespaced resources.
