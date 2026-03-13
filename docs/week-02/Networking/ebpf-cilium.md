# eBPF & Cilium — Kubernetes Networking

## Table of Contents
- [What is eBPF?](#what-is-ebpf)
- [Why eBPF Exists](#why-ebpf-exists)
- [Map Lookup Concept](#map-lookup-concept)
- [Packet Flow Using eBPF](#packet-flow-using-ebpf)
- [What is Cilium?](#what-is-cilium)
- [Kubernetes Networking With Cilium](#kubernetes-networking-with-cilium)
- [iptables vs eBPF](#iptables-vs-ebpf)
- [eBPF Hook Points](#ebpf-hook-points)
- [Security Policies in Cilium](#security-policies-in-cilium)
- [Observability with Hubble](#observability-with-hubble)
- [Example Microservice Traffic](#example-microservice-traffic)
- [Quick Recall](#quick-recall)

---

## What is eBPF?

**eBPF** (Extended Berkeley Packet Filter) is a Linux kernel technology that allows programs to run inside the kernel safely.

### Key Points
- Instead of sending packets to userspace tools, the kernel itself processes networking logic
- eBPF can be used for:
  - Networking
  - Security enforcement
  - Monitoring
  - Observability
  - Tracing

---

## Why eBPF Exists

### Traditional Linux Networking Problem

Traditional Linux networking tools use rule-based processing.

#### Example: iptables
```
packet
  ↓
rule1
rule2
rule3
rule4
...
match
```

### The Problem
- **More rules → slower processing**
- Large Kubernetes clusters may create thousands of rules
- This creates performance bottlenecks

### The Solution
eBPF solves this using **map lookups** (hash tables) instead of sequential rule scanning.

---

## Map Lookup Concept

> **Very Important Concept**

eBPF stores networking data inside **BPF Maps**. Think of this like a HashMap or dictionary.

### Example Map Structure
```
Key                Value
--------------------------------
Service IP         Pod IPs

10.96.0.10 → [10.244.1.5, 10.244.2.7]
```

### How It Works

**When a request arrives:**
```
curl 10.96.0.10
```

**The kernel performs a lookup:**
```
lookup(10.96.0.10) → 10.244.2.7
```

**Result:** Packet is forwarded to the selected pod.

### Why This is Fast
- **HashMap lookup = O(1) time complexity**
- Instead of scanning thousands of rules sequentially
- Constant time regardless of the number of entries

---

## Packet Flow Using eBPF

When a packet arrives, here's the processing flow:

```
Packet
  ↓
Linux Kernel
  ↓
eBPF Program
  ↓
BPF Map Lookup
  ↓
Decision (route / allow / deny)
```

### Example
```
Service IP → Pod IP lookup
```

---

## What is Cilium?

**Cilium** is a Kubernetes CNI (Container Network Interface) that uses eBPF for networking and security.

### What It Replaces
- `kube-proxy`
- `iptables`

### Key Benefit
Traffic routing happens using kernel-level eBPF programs, eliminating the need for traditional userspace tools.

---

## Kubernetes Networking With Cilium

### Traditional Kubernetes Networking
```
Client
  ↓
Service
  ↓
kube-proxy
  ↓
iptables rules
  ↓
Pod
```

### Cilium Networking
```
Client
  ↓
Service
  ↓
eBPF Program
  ↓
BPF Map Lookup
  ↓
Pod
```

**Advantage:** This removes the need for iptables rule scanning, significantly improving performance.

---

## iptables vs eBPF

| Feature | iptables | eBPF |
|---------|----------|------|
| Processing | Rule matching | Map lookup |
| Speed | Slower | Very fast |
| Scalability | Limited | Very high |
| Used by | kube-proxy | Cilium |

### Visual Comparison

**iptables:**
```
packet
  ↓
rule1
rule2
rule3
rule4
...
```

**eBPF:**
```
packet
  ↓
lookup(service_ip)
  ↓
pod
```

---

## eBPF Hook Points

eBPF programs attach to different points in the Linux networking stack.

### Important Hook Points

| Hook | Purpose |
|------|---------|
| **XDP** | Earliest packet processing (at NIC level) |
| **TC** | Traffic control (after kernel stack) |
| **Socket hooks** | Application-level traffic |

### Typical Packet Path
```
Network Card
  ↓
XDP Hook
  ↓
Linux Network Stack
  ↓
TC Hook
  ↓
BPF Map Lookup
  ↓
Pod Network Interface
  ↓
Pod Container
```

---

## Security Policies in Cilium

Cilium can enforce policies at multiple layers.

### Layer 3 Policy (IP Level)

Controls which pods can communicate with which pods.

**Example:**
```
frontend → backend      ✓ allowed
frontend → database     ✗ denied
```

### Layer 4 Policy (Port Level)

Controls which ports are allowed.

**Example:**
```
frontend → backend TCP 80   ✓ allowed
frontend → backend TCP 22   ✗ denied
```

### Layer 7 Policy (Application Level)

Controls application requests at the HTTP/API level.

**Example:**
```
GET    /products   ✓ allowed
POST   /login      ✓ allowed
DELETE /users      ✗ denied
```

**Benefit:** API-level security granularity.

---

## Observability with Hubble

Cilium provides a tool called **Hubble** for real-time network flow visualization.

### Example Output
```
frontend → backend   ✓ allowed
backend → database   ✓ allowed
frontend → database  ✗ denied
```

### Benefits
- Visualize service communication patterns
- Debug network issues
- Security monitoring and compliance

---

## Example Microservice Traffic

### Example Application Architecture
```
User
  ↓
Frontend
  ↓
API Gateway
  ↓
Auth Service
  ↓
Payment Service
  ↓
Database
```

### Example Security Policies
```
Frontend → API Gateway      ✓ allowed
API Gateway → Auth          ✓ allowed
API Gateway → Payment       ✓ allowed
Payment → Database          ✓ allowed
Frontend → Database         ✗ denied
```

### Security Principle
This approach is called **Zero Trust Networking**:
- Nothing is trusted by default
- All connections must be explicitly allowed
- Least privilege principle applied

---

## Quick Recall

### eBPF
- Linux kernel technology
- Fast networking and security
- Uses hash map lookups instead of rule scanning

### Cilium
- Kubernetes CNI using eBPF
- Replaces kube-proxy and iptables
- Enables high-performance, secure networking

### Key Advantage
```
itables → sequential rule scanning (slow)
eBPF     → map lookup O(1) (fast)
```

### Security Layers
- **L3** → IP communication policies
- **L4** → Port-based policies
- **L7** → Application/API request filtering

---

## One-Line Mental Models

> **iptables = rule scanning**  
> **eBPF = hash map lookup**

---

> **Old networking → rule based**  
> **Modern networking → program based**

---

## Interview Summary

| Concept | Description |
|---------|-------------|
| **eBPF** | Kernel technology enabling fast packet processing via hash maps |
| **Cilium** | CNI that leverages eBPF for networking and security policies |
| **Key Advantage** | O(1) map lookups vs O(n) rule matching |
| **Use Cases** | Networking, security, observability, tracing |
| **Replaces** | kube-proxy and iptables |
| **Security Model** | Zero Trust Networking with L3/L4/L7 policies |
| **Observability** | Hubble provides real-time network flow visualization |