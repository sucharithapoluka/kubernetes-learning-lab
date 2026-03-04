This document contains quick recall points useful for Kubernetes interviews.

kubectl Request Flow

When running kubectl get pods command

The following sequence happens internally:
1. `kubectl` reads the **kubeconfig file** which is present in ~/.kube/config path
2. Retrieves the **API server address**
3. Uses the **client certificate and private key** which is present in ~/.kube/config path
4. Sends an **HTTPS request** to the API server
5. API server performs **authentication**
6. **RBAC authorization** checks permissions
7. Data is retrieved from **etcd**
8. Response is returned to `kubectl`

Flow Summary:
=============
kubectl → kubeconfig → API Server → Authentication → Authorization → etcd → Response

Authentication vs Authorization:
===============================
| Concept        | Meaning |
|----------------|--------|
| Authentication | Determines **who the user is** |
| Authorization  | Determines **what actions the user can perform** |

RBAC (Role-Based Access Control) is used for authorization in Kubernetes.


Control Plane vs Worker Nodes:
=============================

Control Plane Components

| Component            | Role |
|----------------------|------|
| kube-apiserver       | Entry point for cluster communication |
| kube-scheduler       | Assigns pods to nodes |
| kube-controller-manager | Maintains desired cluster state |
| etcd                 | Stores cluster configuration and state |

Worker Node Components

| Component | Role |
|----------|------|
| kubelet | Runs containers on the node |
| kube-proxy | Handles service networking |
| Container Runtime | Runs containers (containerd / CRI-O) |

---

when Worker Nodes Come Into Picture
`kubectl` always communicates with the **API Server**.

Pod creation flow:
kubectl
↓
API Server
↓
Scheduler selects a worker node
↓
Kubelet on the worker node creates the pod


Important point:

kubectl **never directly communicates with worker nodes**


Pod Restart Behavior:
=====================
If a pod is managed by a controller such as:

- Deployment
- ReplicaSet
- StatefulSet

Kubernetes automatically recreates the pod when it is deleted.

To stop this behavior, scale the deployment to zero replicas or delete the deploy using below 2 commands

kubectl scale deploy nginx --replicas=0
kubectl delete deploy <container_name> -n <namespace>


Important Networking Concept:
=============================
| Concept | Description |
|-------|-------------|
| Pod Subnet | Range from which pod IPs are assigned |
| Service Subnet | Range used for virtual service IPs |

Pod subnet and service subnet **must never overlap**.


Kubernetes Troubleshooting Basics

Common pod issues:

| Issue | Cause |
|------|------|
| CrashLoopBackOff | Application inside container is crashing |
| ImagePullBackOff | Image cannot be pulled from registry |
| Pending | Scheduler cannot place pod on any node |
| OOMKilled | Container exceeded memory limits |