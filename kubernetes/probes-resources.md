# Kubernetes Probes & Resources

Health checks (probes) and resource management (requests/limits) determine whether your pods are healthy and how they're scheduled. These directly explain many common failures like OOMKilled, restart loops, and Pending pods — essential for the hands-on troubleshooting round.

## Health Probes

Kubernetes uses probes to check container health and decide what to do. There are three types, and knowing the difference is a classic interview question.

| Probe | Question It Answers | On Failure |
|-------|--------------------|-----------| 
| **livenessProbe** | "Is the container alive?" | **Kill and restart** the container |
| **readinessProbe** | "Is it ready for traffic?" | **Remove from Service** (stop sending requests) |
| **startupProbe** | "Has it finished starting?" | **Kill** if startup takes too long |

### The Critical Distinction: Liveness vs Readiness

This is the question interviewers love. Get it crisp:

```
livenessProbe fails   →  Kubernetes RESTARTS the pod
                         "It's broken, reboot it"

readinessProbe fails  →  Kubernetes STOPS SENDING TRAFFIC (but doesn't restart)
                         "It's not ready right now, don't bother it, but it's not broken"
```

**Why the difference matters:**

```
Scenario: Your app is temporarily overwhelmed (busy processing).

If you only had livenessProbe:
  App is slow → liveness fails → K8s KILLS it → makes things WORSE
  (you restarted a healthy-but-busy app, losing its work)

With readinessProbe:
  App is slow → readiness fails → K8s stops NEW traffic → app recovers → 
  readiness passes again → traffic resumes
  (no restart, app drains and recovers gracefully)
```

**Rule of thumb:**
- **Liveness** = "restart me if I'm truly broken/deadlocked"
- **Readiness** = "pause traffic to me while I'm temporarily unable to serve"

### Startup Probe

For **slow-starting applications.** It disables liveness and readiness checks until the app has started, so a slow boot doesn't get killed by an impatient liveness probe.

```
Without startupProbe:
  Slow app takes 60s to start
  livenessProbe checks at 10s → fails → KILLS the app before it ever started
  → CrashLoopBackOff forever

With startupProbe:
  startupProbe allows up to 120s for startup
  Only after startup succeeds do liveness/readiness kick in
  → app boots successfully
```

### Probe Configuration

```yaml
spec:
  containers:
  - name: app
    image: myapp:v1
    ports:
    - containerPort: 8080

    livenessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 15    # wait 15s before first check
      periodSeconds: 10          # check every 10s
      failureThreshold: 3        # 3 failures → restart

    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5

    startupProbe:
      httpGet:
        path: /healthz
        port: 8080
      failureThreshold: 30       # 30 × 10s = 300s max startup time
      periodSeconds: 10
```

### Probe Methods

| Method | How It Checks | Example |
|--------|--------------|---------|
| **httpGet** | HTTP request, 200-399 = healthy | `GET /healthz` |
| **tcpSocket** | TCP connection succeeds | Can connect to port 8080 |
| **exec** | Command exits 0 | `cat /tmp/healthy` |

### Key Probe Parameters

| Parameter | Meaning |
|-----------|---------|
| `initialDelaySeconds` | Wait this long before the first probe |
| `periodSeconds` | How often to probe |
| `timeoutSeconds` | Probe timeout |
| `failureThreshold` | Consecutive failures before acting |
| `successThreshold` | Consecutive successes to be considered healthy |

---

## Resource Requests & Limits

How you tell Kubernetes how much CPU and memory a container needs. This drives **scheduling** and **enforcement** — and directly causes OOMKilled and Pending pods.

```yaml
spec:
  containers:
  - name: app
    image: myapp:v1
    resources:
      requests:          # what K8s RESERVES (guaranteed minimum)
        cpu: "100m"      # 100 millicores = 0.1 CPU
        memory: "256Mi"
      limits:            # the MAXIMUM it can use
        cpu: "500m"      # 0.5 CPU
        memory: "512Mi"
```

### Requests vs Limits

| | Requests | Limits |
|---|----------|--------|
| **Meaning** | Guaranteed minimum (reserved) | Hard maximum (ceiling) |
| **Used for** | **Scheduling** (where the pod fits) | **Enforcement** (capping usage) |
| **If exceeded** | N/A (it's a floor) | CPU: throttled. Memory: **OOMKilled** |

### How Requests Drive Scheduling

```
Node has 1000m CPU, 2Gi memory available.

Pod requests 100m CPU, 256Mi memory.
  → Scheduler checks: does a node have 100m + 256Mi free?
  → Yes → pod scheduled there
  → No node has enough → pod stays PENDING

The scheduler uses REQUESTS (not limits) to decide placement.
```

**This is why pods go Pending:** if no node has enough free resources to satisfy the *requests*, the pod can't be scheduled.

### CPU vs Memory Limits (Crucial Difference)

```
CPU limit exceeded:     Container is THROTTLED (slowed down, but keeps running)
                        → app gets slow, but survives

Memory limit exceeded:  Container is OOMKILLED (terminated)
                        → pod killed, restarted, you see OOMKilled in describe
```

**Memorize this:** CPU is a *compressible* resource (you can throttle it). Memory is *incompressible* (you can't "throttle" RAM — if you're over, you die). That's why exceeding a memory limit kills the container but exceeding a CPU limit just slows it.

### Understanding CPU Units

```
1000m = 1 core (1 full CPU)
500m  = 0.5 core (half a CPU)
100m  = 0.1 core (one tenth of a CPU)

"m" = millicores (thousandths of a core)
```

### Understanding Memory Units

```
Mi = Mebibytes (1024-based): 256Mi = 256 × 1024 × 1024 bytes
M  = Megabytes (1000-based): 256M  = 256 × 1000 × 1000 bytes

Ki, Mi, Gi = binary (1024)
K,  M,  G  = decimal (1000)

Most configs use Mi/Gi.
```

---

## Quality of Service (QoS) Classes

Based on how you set requests and limits, Kubernetes assigns a QoS class that determines **eviction priority** under memory pressure.

| QoS Class | Condition | Eviction Priority |
|-----------|-----------|-------------------|
| **Guaranteed** | requests == limits (both set, equal) | Evicted **last** (safest) |
| **Burstable** | requests < limits (both set) | Evicted **middle** |
| **BestEffort** | no requests or limits set | Evicted **first** (riskiest) |

```
Node runs out of memory → kubelet evicts pods to recover.
Order: BestEffort first, then Burstable, Guaranteed last.
```

**Interview insight:** If you want a critical pod to survive memory pressure, make it **Guaranteed** (set requests == limits). BestEffort pods (no limits) are the first to be killed.

---

## How Probes + Resources Explain Common Failures

This is where it ties together for the troubleshooting round:

```
OOMKilled        → container exceeded its MEMORY LIMIT
                   (fix: raise limit or fix the memory leak)

Pending pod      → no node has enough resources for the REQUESTS
                   (fix: lower requests, add nodes, or free up capacity)

CrashLoopBackOff → container keeps dying; often a livenessProbe killing a
                   slow-starting app, or the app itself crashing
                   (fix: startupProbe for slow starts, or fix the crash)

Not receiving    → readinessProbe is failing, so the Service won't route to it
traffic            (fix: check the readiness endpoint)

CPU throttling   → container hitting its CPU LIMIT, running slow
                   (fix: raise CPU limit)
```

---

## Interview Scenarios

### Scenario 1: "What's the difference between liveness and readiness probes?"

**Strong answer:**
```
"They trigger different actions. If a livenessProbe fails, Kubernetes 
RESTARTS the container — it concludes the app is broken or deadlocked 
and reboots it. If a readinessProbe fails, Kubernetes stops sending 
traffic to the pod but does NOT restart it — the app is temporarily 
unable to serve, so we pause requests and let it recover.

The classic mistake is using only liveness. Imagine an app that's 
temporarily overwhelmed — if liveness fails, K8s kills a 
healthy-but-busy app and makes things worse. With readiness, we just 
stop new traffic, the app drains and recovers, and traffic resumes 
without a restart.

So: liveness = 'restart me if I'm truly broken.' Readiness = 'pause 
traffic while I'm temporarily not ready.'"
```

### Scenario 2: "A pod is OOMKilled. What happened and how do you fix it?"

**Strong answer:**
```
"OOMKilled means the container exceeded its memory LIMIT, so the 
kernel's OOM killer terminated it. Memory is incompressible — unlike 
CPU, you can't throttle it, so going over the limit means death.

I'd confirm with `kubectl describe pod` — the last state will show 
'OOMKilled' with exit code 137. Then I'd figure out if it's:

1. The limit set too low — the app legitimately needs more memory. 
   Fix: raise the memory limit.

2. A memory leak in the app — it grows unbounded. Fix: the limit is 
   actually protecting the node; I need to fix the leak in the code, 
   using `kubectl top pod` and app profiling to confirm the growth.

I'd check the memory trend before deciding. If usage is stable but 
just above the limit, raise it. If it grows forever, it's a leak."
```

### Scenario 3: "A pod is stuck in Pending. Why?"

**Strong answer:**
```
"Pending almost always means the scheduler can't place the pod. The 
most common cause is resources: no node has enough free CPU or memory 
to satisfy the pod's REQUESTS. The scheduler uses requests, not 
limits, for placement.

I'd run `kubectl describe pod` and look at the Events — it'll usually 
say something like 'Insufficient cpu' or 'Insufficient memory,' or 
'0/3 nodes available.'

Other Pending causes: no node matches a nodeSelector/affinity rule, 
an unbound PersistentVolumeClaim, or taints that the pod doesn't 
tolerate. But resource requests are the most common.

Fixes: lower the requests if they're overspecified, add nodes / scale 
the cluster, or free up capacity by removing other workloads."
```

### Scenario 4: "How do requests and limits affect scheduling?"

**Strong answer:**
```
"Requests and limits do different jobs. REQUESTS are what Kubernetes 
reserves and uses for SCHEDULING — the scheduler finds a node with at 
least that much free capacity. LIMITS are the hard ceiling and are 
used for ENFORCEMENT at runtime.

A key consequence: if you set requests too high, pods may go Pending 
because no node has that much free, even if they'd barely use it. Set 
them too low and you risk overcommitting nodes.

And the enforcement differs by resource: exceed a CPU limit and the 
container is throttled (slowed). Exceed a memory limit and it's 
OOMKilled, because memory can't be throttled. That asymmetry is worth 
calling out."
```

---

## Interview Tips

1. **Liveness restarts, readiness removes from traffic** — the #1 probe question
2. **startupProbe is for slow-starting apps** — prevents premature liveness kills
3. **Requests = scheduling, Limits = enforcement** — different jobs
4. **CPU over limit = throttled, Memory over limit = OOMKilled** — the key asymmetry
5. **Pending = scheduler can't fit the requests** — most common cause
6. **Guaranteed QoS (requests == limits) survives eviction longest** — BestEffort dies first
7. **1000m = 1 CPU, Mi = binary megabytes** — know the units

---

## Common Pitfalls

1. **Using only livenessProbe** — can restart healthy-but-busy apps; pair with readiness
2. **No startupProbe for slow apps** — liveness kills them before they boot → CrashLoopBackOff
3. **Thinking CPU limit kills the pod** — CPU throttles; only MEMORY limit kills (OOMKilled)
4. **Setting requests too high** — causes Pending pods even on healthy clusters
5. **No requests/limits at all** — BestEffort QoS, first to be evicted under pressure
6. **Confusing requests and limits** — requests schedule, limits enforce
