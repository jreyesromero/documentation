# Kubernetes Core Concepts

The essential objects that make up a Kubernetes cluster. Know what each one does and how they relate — this is the foundation for both conceptual questions and hands-on troubleshooting.

## The Big Picture

```
┌── CLUSTER ─────────────────────────────────────┐
│                                                │
│   ┌── NAMESPACE ─────────────────────────────┐ │
│   │                                          │ │
│   │   Ingress → Service → Deployment         │ │
│   │                           │              │ │
│   │                      ReplicaSet          │ │
│   │                           │              │ │
│   │                  ┌────────┼────────┐     │ │
│   │                 Pod      Pod      Pod    │ │
│   │                                          │ │
│   │   ConfigMap / Secret → injected → Pods   │ │
│   └──────────────────────────────────────────┘ │
│                                                │
└────────────────────────────────────────────────┘
```

---

## Core Objects

| Object | What It Is | One-Line Mental Model |
|--------|-----------|----------------------|
| **Pod** | Smallest unit; 1+ containers | "A wrapper around your container(s)" |
| **ReplicaSet** | Guarantees N pod replicas | "Keep exactly N copies running" |
| **Deployment** | Manages ReplicaSets + rollouts | "Declarative app management + updates" |
| **Service** | Stable network endpoint for pods | "A load balancer / stable address for pods" |
| **Ingress** | HTTP/HTTPS router (L7) | "The front door for web traffic" |
| **ConfigMap** | Non-secret configuration | "Config files/env vars, in the cluster" |
| **Secret** | Sensitive data | "Like ConfigMap, but for passwords/keys" |
| **Namespace** | Logical isolation | "A virtual cluster within the cluster" |
| **DaemonSet** | One pod per node | "Run this on every node (logging/monitoring)" |
| **StatefulSet** | Pods with identity + storage | "For databases and stateful apps" |

---

## Pod

**The smallest deployable unit in Kubernetes.** A pod wraps one or more containers that share networking and storage.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: app
    image: nginx:1.21
    ports:
    - containerPort: 80
```

**Key facts:**
- A pod usually holds **one** container (occasionally more, e.g. a sidecar)
- Containers in a pod share the same IP and can talk via `localhost`
- Pods are **ephemeral** — they can die and be replaced at any time
- Each pod gets its own IP, but that IP changes when the pod is recreated

**Important:** You rarely create bare Pods directly. You create a **Deployment**, which manages pods for you (restarts them, scales them, updates them).

---

## ReplicaSet

**Ensures a specified number of identical pods are running at all times.**

```
Desired: 3 replicas

If a pod dies:    [pod][pod][ X ]  →  ReplicaSet creates a new one  →  [pod][pod][pod]
If you scale up:  [pod][pod][pod]  →  scale to 5  →  [pod][pod][pod][pod][pod]
```

**You almost never create a ReplicaSet directly** — a Deployment creates and manages it for you. But it's good to know it's the layer that actually maintains the replica count.

---

## Deployment

**The object you actually use to run stateless applications.** It manages ReplicaSets and handles rolling updates.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: web
        image: myapp:v2
        ports:
        - containerPort: 8080
```

**What a Deployment gives you:**
- **Declarative updates** — change the image, K8s rolls out the new version
- **Rolling updates** — gradually replace old pods with new (zero downtime)
- **Rollback** — `kubectl rollout undo` reverts to the previous version
- **Self-healing** — maintains the desired replica count

**The hierarchy:**
```
Deployment  →  manages  →  ReplicaSet  →  manages  →  Pods
(you edit this)            (created automatically)    (the actual containers)
```

---

## Service

**A stable network endpoint for a set of pods.** Since pods are ephemeral (IPs change), a Service gives a consistent address and load-balances across the pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: web-service
spec:
  selector:
    app: web          # routes to pods with label app=web
  ports:
  - port: 80
    targetPort: 8080
```

**The problem Services solve:**
```
Without Service:  Pod IPs change constantly. How do you reach them reliably?
With Service:     Stable DNS name (web-service) + load balancing across pods
```

### Service Types

| Type | What It Does | Use Case |
|------|-------------|----------|
| **ClusterIP** (default) | Internal-only IP | Pod-to-pod communication inside cluster |
| **NodePort** | Exposes on each node's IP at a static port | Simple external access (dev/test) |
| **LoadBalancer** | Provisions a cloud load balancer | Production external access (cloud) |
| **ExternalName** | Maps to an external DNS name | Point to a service outside the cluster |

```
ClusterIP:     only reachable inside the cluster
NodePort:      <NodeIP>:30000-32767 from outside
LoadBalancer:  cloud LB → external IP (AWS ELB, GCP LB, etc.)
```

---

## Ingress

**An HTTP/HTTPS router (Layer 7)** that manages external access to services, typically with host/path-based routing and TLS termination.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-ingress
spec:
  rules:
  - host: app.example.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: api-service
            port:
              number: 80
      - path: /web
        pathType: Prefix
        backend:
          service:
            name: web-service
            port:
              number: 80
```

**What Ingress gives you:**
```
                    ┌──────────────┐
app.example.com  →  │   Ingress    │
                    └──────┬───────┘
                 /api →    │    → /web
                  ┌────────┴────────┐
            api-service        web-service
```

- **Host-based routing:** `api.example.com` vs `web.example.com`
- **Path-based routing:** `/api` → one service, `/web` → another
- **TLS termination:** handles HTTPS certificates
- **Single entry point:** one external IP for many services

**Service vs Ingress:**
- **Service** = L4 (TCP/UDP), internal load balancing, stable IP
- **Ingress** = L7 (HTTP), smart routing by host/path, TLS

**Note:** Ingress requires an **Ingress Controller** (nginx-ingress, Traefik, etc.) to actually do the work. The Ingress object is just the rules.

---

## ConfigMap & Secret

Both inject configuration into pods. The difference is sensitivity.

### ConfigMap (non-sensitive)

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  LOG_LEVEL: "info"
  DATABASE_HOST: "db.internal"
```

### Secret (sensitive)

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: app-secret
type: Opaque
data:
  DB_PASSWORD: cGFzc3dvcmQ=   # base64-encoded
```

**Key difference:**

| | ConfigMap | Secret |
|---|-----------|--------|
| **For** | Config (log levels, URLs, flags) | Passwords, API keys, certs, tokens |
| **Encoding** | Plain text | base64 (NOT encryption!) |
| **Storage** | etcd, plain | etcd, can be encrypted at rest |

**Critical interview point:** Secrets are **base64-encoded, not encrypted** by default. Base64 is trivially reversible (`echo cGFzc3dvcmQ= | base64 -d` → `password`). For real security, you enable encryption-at-rest in etcd or use an external secrets manager (Vault, cloud KMS).

**How they're consumed:** Both can be injected as environment variables or mounted as files in the pod.

---

## Namespace

**A way to logically partition a cluster into virtual sub-clusters.**

```bash
# List namespaces
kubectl get namespaces

# Common defaults:
# default        → where your stuff goes if unspecified
# kube-system    → Kubernetes internal components
# kube-public    → publicly readable
```

**What namespaces are used for:**
- **Isolation** — separate teams, environments (dev/staging), or customers
- **Resource quotas** — limit CPU/memory per namespace
- **RBAC** — scope permissions to a namespace
- **Name scoping** — two namespaces can each have a service named `web`

```
cluster/
├── namespace: team-a      → their own pods, services, configs
├── namespace: team-b      → isolated from team-a
└── namespace: production   → with strict RBAC + quotas
```

**Common pattern (from real experience):** namespace-per-developer or namespace-per-environment, so people don't step on each other.

---

## Workload Types Compared

The three "runs pods differently" objects — a common interview comparison:

| Object | Pods Are... | Use Case |
|--------|------------|----------|
| **Deployment** | Identical, interchangeable, stateless | Web servers, APIs, stateless apps |
| **StatefulSet** | Unique identity, stable name, own storage | Databases, Kafka, anything stateful |
| **DaemonSet** | One per node | Log collectors, monitoring agents, CNI |

### StatefulSet (the stateful one)

```
Deployment pods:   web-7d4f-x8k2, web-7d4f-9j3m  (random names, interchangeable)
StatefulSet pods:  db-0, db-1, db-2              (stable, ordered names + storage)
```

**StatefulSet guarantees:**
- Stable, predictable network identity (`db-0`, `db-1`)
- Stable persistent storage (each pod keeps its own volume across restarts)
- Ordered, graceful deployment and scaling (db-0 before db-1)

**Use when:** The app cares about *which* instance it is (databases, clustered systems, anything where pods aren't interchangeable).

### DaemonSet (the one-per-node one)

```
Node 1: [log-agent]  [your-app]
Node 2: [log-agent]  [your-app]
Node 3: [log-agent]
         ↑
    DaemonSet ensures log-agent runs on EVERY node
```

**Use when:** You need something on every node — log collection (Fluentd), monitoring (node-exporter), networking (CNI plugins).

---

## Interview Scenarios

### Scenario 1: "Explain the relationship between Pod, ReplicaSet, and Deployment"

**Strong answer:**
```
"They're layered. A Pod is the smallest unit — it wraps your 
container(s). But pods are ephemeral; if one dies, it's gone.

A ReplicaSet sits above pods and guarantees a count — 'keep 3 pods 
running.' If one dies, it creates a replacement.

A Deployment sits above the ReplicaSet and adds rollout management — 
declarative updates, rolling deployments, and rollback. When I change 
the image in a Deployment, it creates a new ReplicaSet and gradually 
shifts pods from old to new.

In practice I only ever touch the Deployment. It manages the 
ReplicaSet, which manages the pods. I describe what I want, and the 
layers below make it happen."
```

### Scenario 2: "What's the difference between a Service and an Ingress?"

**Strong answer:**
```
"They operate at different layers. A Service is L4 — it gives a 
stable IP and DNS name for a set of pods and load-balances TCP/UDP 
across them. It solves the problem that pod IPs keep changing.

An Ingress is L7 — it's an HTTP router. It does host- and path-based 
routing, so app.example.com/api goes to one service and /web goes to 
another, all through a single external entry point. It also handles 
TLS termination.

So a Service makes pods reachable with a stable address; an Ingress 
sits in front of Services and routes HTTP traffic intelligently. 
Ingress needs a Service behind it — they work together."
```

### Scenario 3: "When would you use a StatefulSet instead of a Deployment?"

**Strong answer:**
```
"A Deployment treats pods as interchangeable — random names, any pod 
can handle any request, no persistent identity. That's perfect for 
stateless apps like web servers.

A StatefulSet is for when pods are NOT interchangeable — they need 
stable identity and their own persistent storage. Databases are the 
classic case: db-0 and db-1 have stable names, each keeps its own 
volume across restarts, and they start/stop in order.

So: stateless and interchangeable → Deployment. Stateful, with 
identity and persistent storage → StatefulSet. Databases, Kafka, 
Elasticsearch, anything clustered."
```

### Scenario 4: "Are Kubernetes Secrets secure?"

**Strong answer:**
```
"By default, not as much as people assume. Secrets are base64-ENCODED, 
not encrypted. Base64 is trivially reversible — anyone with read access 
can decode them instantly.

What Secrets DO give you over ConfigMaps: they're treated specially 
(not shown in logs by default, can be mounted in memory), and 
Kubernetes can encrypt them at rest in etcd if you enable 
encryption-at-rest.

For real security I'd: enable etcd encryption-at-rest, lock down RBAC 
so only the right pods/users can read them, and for sensitive 
environments use an external secrets manager like Vault or a cloud 
KMS. Base64 encoding alone is not a security boundary."
```

---

## Interview Tips

1. **Know the hierarchy** — Deployment → ReplicaSet → Pod
2. **Pods are ephemeral** — that's why Services and Deployments exist
3. **Service = L4 stable endpoint, Ingress = L7 HTTP router** — common comparison
4. **Secrets are base64, NOT encrypted** — a favorite gotcha question
5. **StatefulSet for stateful** — databases need identity + storage
6. **DaemonSet = one per node** — logging, monitoring, networking agents
7. **You manage Deployments, not Pods directly** — declarative, self-healing

---

## Common Pitfalls

1. **Creating bare Pods** — use Deployments so pods self-heal and update
2. **Thinking Secrets are encrypted** — they're only base64-encoded by default
3. **Confusing Service and Ingress** — L4 endpoint vs L7 HTTP router
4. **Using Deployment for databases** — stateful apps need StatefulSet
5. **Forgetting Ingress needs a controller** — the object alone does nothing
6. **Assuming pod IPs are stable** — they change; that's why Services exist
