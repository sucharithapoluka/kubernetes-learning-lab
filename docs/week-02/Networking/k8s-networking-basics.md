Kubernetes Networking Basics

Kubernetes networking enables communication between pods, services, and external clients.

The goal of Kubernetes networking is:

Every pod gets its own IP address

Pods can communicate directly

Services provide stable access to pods

1. Pod Networking

Each pod gets its own IP address.

Example:

Pod1 → 10.244.1.5
Pod2 → 10.244.2.7

Pods can communicate directly using IP.

Example:

PodA → PodB
10.244.1.5 → 10.244.2.7

This is Layer 3 communication (IP to IP).

2. Kubernetes Networking Rule

Kubernetes has one important networking rule:

Every pod can communicate with every other pod
without NAT.

Meaning:

Pod → Pod communication works directly using IP.

Example:

Pod A (Node1) → Pod B (Node2)

CNI plugins handle this networking.

Examples of CNI:

Flannel
Calico
Cilium
Weave
3. Problem With Pod IPs

Pod IPs are dynamic.

Example:

Pod1 → 10.244.1.5

If the pod restarts:

Old Pod → 10.244.1.5
New Pod → 10.244.3.8

Applications depending on the old IP will break.

So Kubernetes introduces Services.

4. Kubernetes Service

A Service provides a stable virtual IP.

Example:

Service IP → 10.96.0.10

Instead of connecting to pods directly:

Client → Pod IP

Clients connect to the service:

Client → Service IP

The service forwards traffic to backend pods.

Example:

Service → Pod1
Service → Pod2
Service → Pod3
5. Why Services Are Needed

Pods can scale dynamically.

Example:

3 pods today
5 pods tomorrow

Service provides:

Stable IP
Load balancing
Service discovery
6. kube-proxy

kube-proxy runs on every node.

It watches:

Services
Endpoints
EndpointSlices

When a Service is created:

kube-proxy creates iptables rules

These rules handle traffic routing.

7. Important Concept

The Service object does NOT route traffic.

Instead:

kube-proxy installs iptables rules
iptables rules route traffic to pods

Service is just a logical abstraction.

8. Packet Flow (kube-proxy)
Client
 ↓
Node Network Interface
 ↓
Linux Kernel
 ↓
iptables rules (created by kube-proxy)
 ↓
Backend Pod selected
 ↓
Pod receives request

iptables performs load balancing.

9. Example Service Routing

Example service:

Service IP → 10.96.0.10

Backend pods:

10.244.1.5
10.244.2.7
10.244.3.2

iptables rule logic:

if destination == 10.96.0.10
    randomly choose pod

Example result:

10.96.0.10 → 10.244.2.7

Packet is forwarded to that pod.

10. Communication Types in Kubernetes
Pod to Pod
PodA → PodB

Example:

10.244.1.5 → 10.244.2.7

Layer:

Layer 3 (IP communication)
Client to Service
Client → Service IP

Example:

curl 10.96.0.10

Layer:

Layer 4 (IP + Port)

Example:

10.96.0.10:80
HTTP Routing (Ingress)

Example:

GET /products
POST /login

Layer:

Layer 7
11. What Happens Without a Service

Example:

kubectl run nginx --image=nginx

Pod created:

Pod IP → 10.244.1.5

Access:

curl 10.244.1.5

Traffic flow:

Client → Pod IP → Pod

In this case:

kube-proxy does nothing
No service iptables rules created
12. iptables Processing

iptables processes rules sequentially.

Example:

packet
 ↓
rule1
rule2
rule3
...
match

In large clusters:

1000 services
10000 pods

This creates many rules and slows processing.

13. Why eBPF / Cilium Was Introduced

iptables works with rule scanning.

Example:

packet
 ↓
rule1
rule2
rule3
...

eBPF works with map lookup (hash map).

Example:

packet
 ↓
lookup(service_ip)
 ↓
pod

This is much faster.

14. Quick Comparison
Feature	kube-proxy + iptables	eBPF (Cilium)
Routing	iptables rules	BPF maps
Processing	sequential	map lookup
Performance	slower	faster
Scalability	limited	very high
15. Simple Networking Flow

Traditional Kubernetes:

Client
 ↓
Service
 ↓
kube-proxy
 ↓
iptables
 ↓
Pod

Cilium networking:

Client
 ↓
Service
 ↓
eBPF program
 ↓
BPF map lookup
 ↓
Pod
16. Quick Recall (Interview)

Layer meanings:

L3 → IP communication
L4 → Port communication
L7 → Application routing

Kubernetes networking:

Pod → IP
Service → stable virtual IP
kube-proxy → installs iptables rules
iptables → forwards traffic to pods