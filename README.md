# ☸ Kubernetes YAML Collection 🚀

<div align="center">

<img src="https://media.giphy.com/media/LmNwrBhejkK9EFP504/giphy.gif" width="250"/>

# "I deploy therefore I panic." 😵

### Built with YAML, caffeine ☕ and production panic 🚨

![Kubernetes](https://img.shields.io/badge/Kubernetes-YAML%20Madness-blue)
![Status](https://img.shields.io/badge/Cluster-Alive-brightgreen)
![DevOps](https://img.shields.io/badge/Stress-Level-High-red)
![Bugs](https://img.shields.io/badge/Bugs-Expected-yellow)

</div>

---

# 📌 About This Project

This repository contains Kubernetes YAML configuration files for learning and practicing:

* Pods
* Deployments
* ReplicaSets
* Services
* Persistent Volumes
* Persistent Volume Claims
* DaemonSets
* Jobs
* Load Balancers

```bash
kubectl apply -f confidence.yml
```

---

# 📂 Project Structure

```bash
.
├── Load-Balancer/
├── DaemonSet.yml
├── Deployment.yml
├── Deployment-2.yml
├── Deployment-Mounting.yml
├── Job.yml
├── NodePort.yml
├── PersistemVolume.yml
├── PersistentVolumeClaim.yml
├── ReplicaSet.yml
├── ReplicaSet-1.yml
├── ReplicaSet-2.yml
├── SVC-ClusterIP.yml
├── pod.yml
├── pod-1.yml
└── README.md
```

---

# ☸ Kubernetes Resources Included

| File                        | Description                       |
| --------------------------- | --------------------------------- |
| `pod.yml`                   | Basic Pod Example                 |
| `Deployment.yml`            | Kubernetes Deployment             |
| `Deployment-2.yml`          | Additional Deployment Example     |
| `Deployment-Mounting.yml`   | Deployment with Volume Mount      |
| `ReplicaSet.yml`            | ReplicaSet Configuration          |
| `ReplicaSet-1.yml`          | ReplicaSet Example 1              |
| `ReplicaSet-2.yml`          | ReplicaSet Example 2              |
| `NodePort.yml`              | Expose Application Using NodePort |
| `SVC-ClusterIP.yml`         | Internal Cluster Communication    |
| `PersistemVolume.yml`       | Persistent Volume Configuration   |
| `PersistentVolumeClaim.yml` | PVC Example                       |
| `DaemonSet.yml`             | Run Pods on Every Node            |
| `Job.yml`                   | One-Time Task Execution           |
| `Load-Balancer/`            | LoadBalancer Configurations       |

---

# 🚀 Getting Started

## 1️⃣ Clone Repository

```bash
git clone https://github.com/your-username/kubernetes-yaml-collection.git
```

---

## 2️⃣ Move Into Project

```bash
cd kubernetes-yaml-collection
```

---

## 3️⃣ Apply YAML Files

Apply a single file:

```bash
kubectl apply -f pod.yml
```

Apply all YAML files:

```bash
kubectl apply -f .
```

---

# 🔥 Useful Kubernetes Commands

## Check Pods

```bash
kubectl get pods
```

---

## Check Services

```bash
kubectl get svc
```

---

## Check Deployments

```bash
kubectl get deployments
```

---

## Describe Resources

```bash
kubectl describe pod <pod-name>
```

---

## View Logs

```bash
kubectl logs <pod-name>
```

---

# ⚠ Kubernetes Reality Check

## What We Expect

```bash
Everything Running ✅
```

## What Actually Happens

```bash
CrashLoopBackOff ❌
ImagePullBackOff ❌
Pending ❌
```

---

# 🧠 DevOps Wisdom

```python
while True:
    try:
        kubectl_apply()
        break
    except:
        panic()
```

---

# 🔥 Troubleshooting Guide

If something breaks:

```bash
kubectl delete pod --all
```

Still broken?

```bash
Restart cluster
```

Still broken?

```bash
Blame YAML indentation
```

---

# 📊 Project Status

| Component               | Status     |
| ----------------------- | ---------- |
| Pods                    | 🟢 Running |
| Services                | 🟢 Active  |
| Volumes                 | 🟡 Maybe   |
| Load Balancer           | 🟢 Working |
| Developer Mental Health | 🔴 Missing |

---

# 💀 Famous Last Words

```bash
"It was working yesterday."
```

---

# 🤝 Contribution

Contributions are welcome.

If you understand Kubernetes completely,
please teach the rest of us 😭

---

# ⭐ Support

If this repository helped you:

⭐ Star the repository

If this repository caused suffering:

😅 Welcome to DevOps

---

<div align="center">

# ☸ My Pods Are Running.

# I Am Not.

### Made with ❤️ and YAML indentation errors.

</div>
