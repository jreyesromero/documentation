# Kubernetes Architecture - SRE Interview Guide

**Time to master:** 40 minutes  
**Difficulty:** ⭐⭐⭐ (Medium)  
**Interview frequency:** Very High (foundational question in most technical rounds)

---

## ⚡ Quick Summary

Kubernetes has two main parts:
- **Control Plane** (brain) - Makes decisions, stores state
- **Worker Nodes** (workers) - Run actual containers

System components run in `kube-system` namespace. **Metrics Server** collects cluster metrics for monitoring and autoscaling.

---

## 🎯 The Architecture at a Glance

```
┌──────────────────── KUBERNETES CLUSTER ──────────────────────┐
│                                                               │
│  ┌────────────────── CONTROL PLANE (Master) ────────────┐   │
│  │                                                       │   │
│  │                    ┌──────────────┐                  │   │
│  │                    │  API Server  │                  │   │
│  │                    │ (Coordinator)│                  │   │
│  │                    └──────────────┘                  │   │
│  │                         ↑↓                           │   │
│  │                    (reads/writes)                    │   │
│  │                         ↑↓                           │   │
│  │                    ┌──────────────┐                  │   │
│  │                    │     etcd     │                  │   │
│  │                    │  (Database)  │                  │   │
│  │                    └──────────────┘                  │   │
│  │                                                       │   │
│  │    ┌─────────────────┬──────────────────┐           │   │
│  │    ↓                 ↓                  ↓           │   │
│  │  Scheduler      Controller Manager   Metrics Server │   │
│  │  (Assign pods)  (Reconcile state)    (Collect CPU/  │   │
│  │                                        memory)       │   │
│  │                                                       │   │
│  └───────────────────────────────────────────────────────┘   │
│                         ↓                                     │
│              (API server coordinates all)                     │
│                         ↓                                     │
│  ┌────────────── WORKER NODES (can be many) ──────────┐    │
│  │                                                     │    │
│  │  ┌─ Node 1 ─┐  ┌─ Node 2 ─┐  ┌─ Node N ─┐        │    │
│  │  │ Kubelet   │  │ Kubelet   │  │ Kubelet   │        │    │
│  │  │ kube-     │  │ kube-     │  │ kube-     │        │    │
│  │  │ proxy     │  │ proxy     │  │ proxy     │        │    │
│  │  │ Container │  │ Container │  │ Container │        │    │
│  │  │ Runtime   │  │ Runtime   │  │ Runtime   │        │    │
│  │  │ Pods...   │  │ Pods...   │  │ Pods...   │        │    │
│  │  └───────────┘  └───────────┘  └───────────┘        │    │
│  │       ↑              ↑              ↑                │    │
│  │  (Metrics collected by Metrics Server)              │    │
│  │                                                     │    │
│  └─────────────────────────────────────────────────────┘    │
│                                                               │
└──────────────────────────────────────────────────────────────┘
```

---

## 🏗️ Control Plane Components

### 1. **API Server** (`kube-apiserver`)
- **Role:** The front door of Kubernetes
- **What it does:** 
  - Accepts all kubectl commands
  - Validates requests
  - Stores/retrieves data from etcd
  - Authenticates and authorizes users

```bash
# When you run: kubectl create deployment web --image=nginx
# The API Server:
# 1. Receives the request
# 2. Authenticates you (who are you?)
# 3. Authorizes (do you have permission?)
# 4. Validates (is this valid YAML/object?)
# 5. Stores in etcd
# 6. Returns success
```

**Interview questions:**
- "What happens when you kubectl apply a deployment?"
- "Where does the API server store data?"
- "How does the API server talk to workers?"

---

### 2. **etcd** (database)
- **Role:** The memory of the cluster
- **What it does:**
  - Stores ALL cluster state (deployments, services, configs, secrets)
  - Consistent, replicated key-value store
  - Only the API server should directly access it

```bash
# What's stored in etcd?
# - All Deployments, Pods, Services
# - Secrets, ConfigMaps
# - Node information
# - RBAC policies
# - Literally everything about your cluster

# Example: When you create a Deployment
kubectl create deployment web --image=nginx
# etcd now stores:
# /registry/deployments/default/web = {deployment spec}
```

**Critical for SRE:**
- **Backup etcd regularly** (disaster recovery)
- **Encrypt etcd data** (security for Secrets)
- **Etcd goes down = cluster paralyzed** (can't create/delete anything)

**Interview questions:**
- "What happens if etcd is corrupted?"
- "How do you backup a Kubernetes cluster?"
- "Where are Secrets stored?"

---

### 3. **Scheduler** (`kube-scheduler`)
- **Role:** The matchmaker
- **What it does:**
  - Watches for new Pods without a node assignment
  - Picks the best node based on:
    - Resource requests (CPU, memory)
    - Node affinity rules
    - Taints and tolerations
    - Custom scheduling policies

```bash
# When you create a Pod:
# 1. Pod created in etcd (no node assigned yet)
# 2. Scheduler sees the unassigned Pod
# 3. Scheduler evaluates all nodes
# 4. Finds best fit (most resources available, affinity rules, etc.)
# 5. Updates Pod with node assignment
# 6. Kubelet on that node picks it up and runs it

# What scheduler looks at:
# - Pod resource requests (memory, CPU)
# - Node available resources
# - Node labels and taints
# - Pod affinity/anti-affinity rules
```

**Interview questions:**
- "Why didn't my pod schedule?"
- "Scheduler uses requests or limits?"
- "How do you force a pod to specific nodes?"

---

### 4. **Controller Manager** (`kube-controller-manager`)
- **Role:** The overseer - ensures desired state matches actual state
- **What it does:**
  - Runs multiple controllers that reconcile state
  - If a Deployment says "3 replicas", ensures exactly 3 pods exist
  - If a pod dies, controller notices and asks scheduler to create a new one

```bash
# Key controllers:
# - Deployment Controller: watches Deployments, creates/updates ReplicaSets
# - ReplicaSet Controller: watches ReplicaSets, creates/deletes Pods
# - Node Controller: watches nodes, marks unhealthy ones
# - Service Account Controller: creates default service accounts

# Example flow:
# You: kubectl create deployment web --replicas=3
# etcd: stores Deployment spec
# Deployment Controller: sees Deployment, creates ReplicaSet
# ReplicaSet Controller: sees ReplicaSet, creates 3 Pods
# Scheduler: assigns Pods to nodes
# Kubelets: run the Pods

# If a Pod dies:
# ReplicaSet Controller: notices (one pod missing)
# Creates replacement Pod immediately
# Deployment always maintains desired state
```

**Interview questions:**
- "How does Deployment create Pods?"
- "What happens if a pod crashes?"
- "Why does my Deployment keep creating new pods?"

---

## 💻 Worker Node Components

### 1. **Kubelet** (on each node)
- **Role:** The node's manager
- **What it does:**
  - Talks to API server (watches for pods assigned to it)
  - Manages containers on this node
  - Runs probes (liveness, readiness)
  - Reports node status back to API server

```bash
# Kubelet job:
# 1. Watch API server for pods assigned to this node
# 2. Pull container images
# 3. Start containers via container runtime
# 4. Monitor pod health
# 5. Run liveness/readiness probes
# 6. Report back: "Pod is running" or "Pod crashed"
# 7. Delete pods when told to stop
```

**Interview questions:**
- "Which component runs the pod?"
- "Who runs the liveness probe?"
- "What talks to the API server from worker nodes?"

---

### 2. **kube-proxy** (on each node)
- **Role:** The network manager
- **What it does:**
  - Implements Services (load balancing)
  - Sets up iptables rules to route traffic to pods
  - Handles ClusterIP load balancing

```bash
# When you create a Service:
Service Name: my-service
Type: ClusterIP
Port: 80
Selector: app=web
Endpoints: 10.0.1.5, 10.0.1.6, 10.0.1.7

# kube-proxy on EVERY node:
# 1. Sees the Service
# 2. Creates iptables rule: traffic to 10.96.0.1:80 → round-robin to endpoints
# 3. Now any pod can curl http://my-service:80 from any node

# Result: Load balancing across all pods matching the selector
```

**Interview questions:**
- "How does a Service distribute traffic?"
- "What's a ClusterIP?"
- "Why do you need kube-proxy?"

---

### 3. **Container Runtime** (Docker, containerd, etc.)
- **Role:** Actually runs the containers
- **What it does:**
  - Pulls images
  - Creates containers
  - Starts/stops containers
  - Manages container resources

---

## 📊 System Namespaces

**Where do these components actually run?**

```bash
# Control plane components (usually master nodes):
kubectl get pods -n kube-system
# Shows:
# kube-apiserver-master-1
# kube-scheduler-master-1
# kube-controller-manager-master-1
# etcd-master-1
# coredns-1234567890-abc (DNS)
# metrics-server-1234567890-def (METRICS!)

# Other system namespaces:
kubectl get ns
# kube-system     (core components, metrics-server)
# kube-public     (cluster info, shared data)
# kube-node-lease (node heartbeats)
# default         (your apps)
```

**Key fact:** By default, most system components are in `kube-system` namespace.

---

## 📈 Metrics Server (THE ANSWER TO "What gets cluster metrics?")

### **Metrics Server** - The metrics collector

**Role:** Collects performance metrics from the cluster

**What it does:**
- Periodically scrapes metrics from kubelets on every node
- Collects: CPU usage, memory usage per pod
- Makes metrics available via Metrics API
- Used by `kubectl top` and Horizontal Pod Autoscaler (HPA)

```bash
# Install metrics-server (if not already there):
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml

# View metrics:
kubectl top nodes          # CPU/memory per node
kubectl top pods           # CPU/memory per pod
kubectl top pods -n my-ns  # In specific namespace

# Example output:
# NAME     CPU(cores)   MEMORY(bytes)
# node-1   512m         2048Mi
# node-2   256m         1024Mi

# This data is used by:
# - kubectl top (manual inspection)
# - HPA (Horizontal Pod Autoscaler - auto-scale based on CPU/memory)
# - Custom monitoring dashboards
```

**Why this matters for SRE:**
- **Debug resource issues:** `kubectl top pods` shows what's actually using resources
- **Autoscaling:** HPA scales pods based on metrics from Metrics Server
- **Capacity planning:** See what your cluster is actually using

**Interview questions:**
- "What component collects cluster metrics?"
- "How does HPA know when to scale?"
- "Why is `kubectl top` empty?" (Usually means Metrics Server not installed)

---

## 🔄 How Everything Works Together (Full Flow)

```
You run: kubectl create deployment web --replicas=3 --image=nginx

1. API Server receives request
   ├─ Authenticates you
   ├─ Validates request
   └─ Stores in etcd: Deployment object

2. Deployment Controller watches etcd
   └─ Sees new Deployment
   └─ Creates ReplicaSet (desired: 3 pods)
   └─ Stores ReplicaSet in etcd

3. ReplicaSet Controller watches etcd
   └─ Sees new ReplicaSet
   └─ Creates 3 Pod objects
   └─ Stores Pods in etcd (with no node assignment yet)

4. Scheduler watches etcd
   └─ Sees 3 unassigned Pods
   └─ For each Pod:
      ├─ Evaluates all nodes
      ├─ Picks best fit (resources, affinity, etc.)
      └─ Updates Pod with node assignment
   └─ Stores assignment in etcd

5. Kubelet on each assigned node watches etcd
   └─ Sees Pod assigned to it
   └─ Pulls nginx image
   └─ Starts container via container runtime
   └─ Reports status: "Pod is running"
   └─ Stores status in etcd

6. kube-proxy on every node watches for Services
   └─ When you create a Service pointing to these pods
   └─ Creates iptables rules
   └─ Result: traffic to Service → load balanced to pods

7. Metrics Server periodically collects metrics
   └─ Asks kubelets: "How much CPU/memory is each pod using?"
   └─ Stores in short-term cache
   └─ Available via: kubectl top pods, Metrics API, HPA

Result: Your 3 nginx pods are running, load balanced, and monitored!
```

---

## ❓ Typical Interview Questions (Real Examples)

### Q1: "What are the main components of Kubernetes architecture?"

**Answer:**
```
"Kubernetes has two main parts:

Control Plane (brain):
- API Server: Entry point, validates requests, stores state
- etcd: Database storing all cluster state
- Scheduler: Assigns pods to nodes based on resources
- Controller Manager: Ensures desired state matches actual

Worker Nodes (workers):
- Kubelet: Runs pods, monitors health, talks to API server
- kube-proxy: Implements Services, load balancing
- Container Runtime: Actually runs containers

These run in kube-system namespace on master nodes.
Worker nodes run user pods (in other namespaces)."
```

---

### Q2: "What component collects metrics from the cluster?"

**Answer:**
```
"Metrics Server.

It runs in kube-system namespace and:
- Scrapes metrics from kubelets on every node
- Collects CPU and memory usage
- Exposes via Metrics API
- Used by: kubectl top, HPA (Horizontal Pod Autoscaler)

If kubectl top is empty, Metrics Server likely isn't installed:
kubectl get deployment metrics-server -n kube-system
"
```

---

### Q3: "Walk me through what happens when you create a Deployment with 3 replicas"

**Answer (showing understanding of all components):**
```
1. API Server validates your kubectl create command
   - Stores Deployment in etcd

2. Deployment Controller sees the Deployment
   - Creates a ReplicaSet with 3 replicas desired

3. ReplicaSet Controller sees the ReplicaSet
   - Creates 3 Pod objects

4. Scheduler sees 3 unassigned Pods
   - Evaluates nodes based on:
     * Resource requests vs available resources
     * Node affinity/taints
     * Pod affinity rules
   - Assigns each Pod to best-fit node

5. Kubelet on each assigned node sees its Pod
   - Pulls the image
   - Starts container
   - Runs liveness/readiness probes
   - Reports status back to API server

6. If a pod crashes:
   - Kubelet detects (liveness probe fails)
   - Reports to API server
   - ReplicaSet Controller notices (only 2 running)
   - Requests new pod creation
   - Scheduler assigns it
   - Kubelet runs it

7. If node dies:
   - Node Controller notices no heartbeat
   - Marks node as NotReady
   - Controller Manager evicts pods
   - Scheduler creates replacements on healthy nodes
```

---

### Q4: "What's stored in etcd and why is it critical?"

**Answer:**
```
"Everything about the cluster is in etcd:
- Deployments, StatefulSets, DaemonSets
- Pods, Services, Ingresses
- ConfigMaps and Secrets
- Nodes, Namespaces
- RBAC policies
- Literally the entire cluster state

Why critical:
- Only the API server should access it
- If etcd is down, cluster is paralyzed
  (can't create/delete/update anything)
- If etcd is corrupted, cluster is corrupted
- MUST back it up regularly for disaster recovery
- Secrets stored here should be encrypted at rest
"
```

---

## 🚨 Common Mistakes

❌ **"etcd runs on worker nodes"**  
✅ etcd runs on control plane (master) only

❌ **"Kubelet runs in a container"**  
✅ Kubelet runs as a system daemon directly on the node

❌ **"The scheduler runs pods"**  
✅ Scheduler assigns pods to nodes; Kubelet actually runs them

❌ **"Metrics Server is optional"**  
✅ Needed for `kubectl top` and HPA; often not installed by default

❌ **"Control plane components run in default namespace"**  
✅ They run in `kube-system` namespace

---

## 📂 Component Locations

```bash
# View control plane components
kubectl get pods -n kube-system

# View system namespaces
kubectl get ns

# Specifically check Metrics Server
kubectl get deployment -n kube-system metrics-server

# View where components are running
kubectl get pods -n kube-system -o wide

# Check node status (Node Controller updates this)
kubectl get nodes

# View cluster info (stored in kube-public namespace)
kubectl cluster-info
```

---

## 🎯 Interview Prep Checklist

- [ ] Can explain control plane vs worker nodes
- [ ] Know 4 control plane components by heart
- [ ] Know 2 worker node components by heart
- [ ] Can explain what etcd is and why it matters
- [ ] Understand the flow: create → API → etcd → Controller → Scheduler → Kubelet
- [ ] Know Metrics Server gets cluster metrics
- [ ] Can name system namespaces (kube-system, kube-public, kube-node-lease)
- [ ] Understand why control plane going down = cluster paralyzed
- [ ] Know how to check if Metrics Server is installed

---

## 🔗 Related Files

- [Concepts](./concepts.md) - Kubernetes objects (Pod, Deployment, Service, etc.)
- [Probes & Resources](./probes-resources.md) - Health checks and resource management
- [kubectl Commands](./kubectl-commands.md) - Debugging tools
- [Troubleshooting](./troubleshooting.md) - Common failure states

---

**Next:** Review actual component logs with `kubectl logs -n kube-system` to see them in action.
