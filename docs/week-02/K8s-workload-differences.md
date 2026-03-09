# Kubernetes Workloads: Deployment vs StatefulSet vs DaemonSet

## Overview

Kubernetes provides different workload controllers to manage pods depending on the application behavior.

The three most common controllers are:

* **Deployment**
* **StatefulSet**
* **DaemonSet**

---

# 1. Deployment

Used mainly for **stateless applications**.

Examples:

* Web applications
* REST APIs
* Frontend services

### Characteristics

* Pods are **interchangeable**.
* Pod identity **does not matter**.
* Pod names are **randomly generated**.
* Supports **scaling and rolling updates**.

Example pod names:

```
nginx-deployment-6f7c8d9f5b-x7k9m
nginx-deployment-6f7c8d9f5b-p2r8q
```

---

# 2. StatefulSet

Used for **stateful applications** where pod identity and storage matter.

Examples:

* MySQL
* PostgreSQL
* MongoDB
* Kafka

### Characteristics

* Pods have **stable identity**.
* Pods are created in **ordered sequence**.
* Each pod can have **its own persistent volume**.
* Pod names are **predictable**.

Example pod names:

```
mysql-0
mysql-1
mysql-2
```

---

# 3. DaemonSet

Used when a pod must run on **every matching node**.

Examples:

* kube-proxy
* Fluentd (log collection)
* Node exporter
* CNI agents

### Characteristics

* **One pod per node**.
* Automatically runs on **new nodes**.
* Typically used for **node-level services**.

---

# Key Differences

| Feature      | Deployment     | StatefulSet                | DaemonSet                |
| ------------ | -------------- | -------------------------- | ------------------------ |
| Main use     | Stateless apps | Stateful apps              | One pod per node         |
| Pod identity | Not stable     | Stable                     | Node based               |
| Pod naming   | Random         | Ordered (`app-0`)          | Per node                 |
| Storage      | Optional       | Per‑pod persistent storage | Usually not required     |
| Scaling      | By replicas    | By replicas (ordered)      | Based on number of nodes |
| Example apps | Nginx, APIs    | Databases, Kafka           | kube-proxy, monitoring   |

---

# Easy Way to Remember

```
Deployment  -> Stateless apps
StatefulSet -> Stateful apps with identity
DaemonSet   -> One pod per node
```

---

# Useful Commands

## Deployment

```bash
kubectl get deployment
kubectl describe deployment <name>
kubectl scale deployment <name> --replicas=3
kubectl rollout restart deployment <name>
```

## StatefulSet

```bash
kubectl get statefulset
kubectl describe statefulset <name>
kubectl scale statefulset <name> --replicas=3
kubectl rollout restart statefulset <name>
```

## DaemonSet

```bash
kubectl get daemonset
kubectl describe daemonset <name>
kubectl rollout restart daemonset <name>
```

Example:

```bash
kubectl get daemonset kube-proxy -n kube-system -o yaml
```
