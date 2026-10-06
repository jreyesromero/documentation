# Kubernetes Troubleshooting

The hands-on debugging guide. Each common failure state mapped to its causes, diagnosis, and fix. This is the core material for the hands-on Kubernetes troubleshooting interview round.

## The Universal Debugging Flow

For any pod problem, start here:

```
kubectl get pods              → What's the status?
kubectl describe pod <name>   → Why? (check Last State + Events)
kubectl logs <name> --previous → What did the crashed app say?
kubectl get events            → Any cluster-level issues?
```

---

## Pod Status Cheat Sheet

| Status | Meaning | Jump To |
|--------|---------|---------|
| **CrashLoopBackOff** | Container keeps crashing and restarting | [Section 1](#1-crashloopbackoff) |
| **ImagePullBackOff / ErrImagePull** | Can't pull the container image | [Section 2](#2-imagepullbackoff--errimagepull) |
| **Pending** | Can't be scheduled onto a node | [Section 3](#3-pending) |
| **OOMKilled** | Exceeded memory limit, killed | [Section 4](#4-oomkilled) |
| **Running but not Ready (0/1)** | Readiness probe failing | [Section 5](#5-running-but-not-ready) |
| **Terminating (stuck)** | Won't finish deleting | [Section 6](#6-stuck-terminating) |
| **CreateContainerConfigError** | Missing ConfigMap/Secret | [Section 7](#7-createcontainerconfigerror) |

---

## 1. CrashLoopBackOff

**The container starts, crashes, restarts, crashes again** — in a loop. Kubernetes adds increasing delay ("back-off") between restarts.

### Diagnosis

```bash
# 1. Confirm
kubectl get pods
# web-9j3m   0/1   CrashLoopBackOff   8   12m

# 2. Why did it last die?
kubectl describe pod web-9j3m
# Look at: Last State → Reason + Exit Code

# 3. THE KEY STEP — logs from the crashed container
kubectl logs web-9j3m --previous
# This shows the actual error/stack trace
```

### Common Causes & Exit Codes

| Exit Code | Meaning | Likely Cause |
|-----------|---------|--------------|
| **0** | Clean exit | App ran and exited (maybe not meant to be long-running) |
| **1** | General error | Application error/exception on startup |
| **137** | SIGKILL (128+9) | **OOMKilled** (out of memory) or force-killed |
| **139** | SIGSEGV (128+11) | Segmentation fault (bug/crash) |
| **143** | SIGTERM (128+15) | Graceful termination |

### Root Causes & Fixes

```
1. APPLICATION ERROR ON STARTUP
   → logs --previous shows the exception
   → Fix: the bug, bad config, missing dependency

2. MISSING CONFIG / ENV VAR / SECRET
   → App crashes because it can't find a required value
   → Fix: check ConfigMap/Secret is present and mounted

3. LIVENESS PROBE KILLING A SLOW-STARTING APP
   → App needs 60s to boot, liveness checks at 10s → kills it
   → Fix: add a startupProbe, or raise initialDelaySeconds

4. CAN'T CONNECT TO A DEPENDENCY
   → App needs a database that's unreachable → crashes on startup
   → Fix: check the dependency (Service, DNS, network policy)

5. OOMKilled (exit 137)
   → See Section 4
```

### Debugging Script

```bash
POD=web-9j3m

# The crash reason
kubectl describe pod $POD | grep -A5 "Last State"

# The actual error
kubectl logs $POD --previous

# Is it a config issue?
kubectl describe pod $POD | grep -A10 "Events"

# Is it a slow-start issue?
kubectl describe pod $POD | grep -A3 "Liveness"
```

---

## 2. ImagePullBackOff / ErrImagePull

**Kubernetes can't pull the container image.** It retries with back-off.

### Diagnosis

```bash
kubectl describe pod <name>
# Events will show the exact reason:
#   Failed to pull image "myapp:v2": ... not found
#   Failed to pull image: unauthorized
```

### Common Causes & Fixes

| Cause | Event Message | Fix |
|-------|--------------|-----|
| **Typo in image name/tag** | `manifest for myapp:v2 not found` | Fix the image name/tag |
| **Image doesn't exist** | `not found` | Build/push the image, check the tag |
| **Private registry, no auth** | `unauthorized` / `pull access denied` | Add an imagePullSecret |
| **Wrong registry** | `no such host` | Fix the registry URL |
| **Rate limited (Docker Hub)** | `toomanyrequests` | Authenticate or use a mirror |

### Fixes

```bash
# Typo — check the actual image reference
kubectl get pod <name> -o jsonpath='{.spec.containers[*].image}'
# myapp:v2  ← is that tag correct? Does it exist in the registry?

# Private registry auth — create and reference a pull secret
kubectl create secret docker-registry regcred \
  --docker-server=registry.example.com \
  --docker-username=user \
  --docker-password=pass

# Then reference it in the pod spec:
#   spec:
#     imagePullSecrets:
#     - name: regcred

# Verify the image exists (from your machine)
docker pull myapp:v2
```

---

## 3. Pending

**The pod can't be scheduled onto any node.** It has no node to run on.

### Diagnosis

```bash
kubectl describe pod <name>
# Events show WHY the scheduler can't place it:
#   0/3 nodes available: 3 Insufficient memory
#   0/3 nodes available: 3 node(s) didn't match node selector
#   0/3 nodes available: 3 node(s) had taint {...}
```

### Common Causes & Fixes

| Cause | Event Message | Fix |
|-------|--------------|-----|
| **Insufficient resources** | `Insufficient cpu/memory` | Lower requests, add nodes, free capacity |
| **Node selector / affinity** | `didn't match node selector` | Fix the selector or label a node |
| **Taints** | `had taint {...} that the pod didn't tolerate` | Add a toleration or remove the taint |
| **Unbound PVC** | `pod has unbound PersistentVolumeClaim` | Check the PVC / StorageClass |
| **No nodes at all** | `no nodes available` | Cluster has no (ready) nodes |

### The Most Common: Insufficient Resources

```bash
# The pod requests more than any node has free
kubectl describe pod <name> | grep -A3 Requests
#   Requests:
#     cpu:     2
#     memory:  4Gi

# Check what nodes have available
kubectl describe nodes | grep -A5 "Allocated resources"

# Check node capacity
kubectl top nodes

# Fixes:
# 1. Lower the request if overspecified
# 2. Scale the cluster (add nodes)
# 3. Free capacity (remove/scale down other workloads)
```

---

## 4. OOMKilled

**The container exceeded its memory limit and was killed** by the kernel's OOM (Out Of Memory) killer. Exit code **137**.

### Diagnosis

```bash
kubectl describe pod <name>
# Containers:
#   app:
#     Last State:  Terminated
#       Reason:    OOMKilled          ← here
#       Exit Code: 137
#     Limits:
#       memory:    256Mi              ← the limit it hit
```

### Is It a Bad Limit or a Memory Leak?

This is the key diagnostic question:

```bash
# Watch memory usage over time
kubectl top pod <name>

# Scenario A: Usage is stable, just above the limit
#   → Limit is too low. The app legitimately needs more.
#   → Fix: raise the memory limit.

# Scenario B: Usage grows continuously until OOMKill
#   → Memory LEAK in the application.
#   → The limit is actually protecting the node.
#   → Fix: fix the leak in the code (raising the limit just delays it).
```

### Fixes

```bash
# If the limit is genuinely too low — raise it
# Edit the Deployment:
#   resources:
#     limits:
#       memory: 512Mi      # was 256Mi

kubectl set resources deployment <name> --limits=memory=512Mi

# If it's a leak — the limit is doing its job (protecting the node).
# Fix the application. Monitor with:
kubectl top pod <name> --sort-by=memory
```

**Interview insight:** Don't just raise the limit reflexively. First determine *why* it's hitting the limit. If it's a leak, raising the limit only delays the inevitable and risks taking down the whole node.

---

## 5. Running but Not Ready

**Pod shows `Running` but `0/1 READY`.** The container is up, but its **readinessProbe is failing**, so the Service won't send it traffic.

### Diagnosis

```bash
kubectl get pods
# web-x8k2   0/1   Running   0   5m      ← Running but 0/1 ready

kubectl describe pod web-x8k2
# Events:
#   Warning  Unhealthy  Readiness probe failed: HTTP probe failed with statuscode: 503
```

### Common Causes & Fixes

```
1. APP NOT ACTUALLY READY YET
   → Still loading data, warming cache, connecting to DB
   → Fix: adjust readiness initialDelaySeconds, or it's working as intended

2. READINESS ENDPOINT WRONG
   → Probe checks /ready but app serves /health
   → Fix: correct the probe path/port

3. DEPENDENCY DOWN
   → Readiness returns 503 because the app can't reach its database
   → Fix: the dependency; readiness is correctly reporting "not ready"

4. PROBE TOO STRICT
   → timeout too short, app responds slowly
   → Fix: increase timeoutSeconds / failureThreshold
```

### Debugging

```bash
# Check the readiness config
kubectl describe pod web-x8k2 | grep -A5 Readiness

# Test the readiness endpoint from inside the pod
kubectl exec -it web-x8k2 -- curl localhost:8080/ready
# See what it actually returns

# Check app logs for why it's not ready
kubectl logs web-x8k2
```

**Why it matters:** A pod that's never Ready gets no traffic. If ALL pods behind a Service are not-ready, the Service has no endpoints and requests fail — even though pods are "Running."

---

## 6. Stuck Terminating

**A pod stays in `Terminating` state and won't delete.**

### Diagnosis

```bash
kubectl get pods
# web-x8k2   1/1   Terminating   0   10m    ← stuck

kubectl describe pod web-x8k2
# Look for finalizers, volume issues
```

### Common Causes & Fixes

```
1. FINALIZERS
   → A finalizer is blocking deletion (waiting for cleanup)
   → Check: kubectl get pod <name> -o yaml | grep finalizers
   → Fix: resolve what the finalizer waits for, or remove it (carefully)

2. GRACEFUL SHUTDOWN TAKING TOO LONG
   → App ignoring SIGTERM, terminationGracePeriod not elapsed
   → Fix: wait, or the app isn't handling SIGTERM properly

3. NODE UNREACHABLE
   → The node died; K8s can't confirm the pod is gone
   → Fix: the node, or force-delete

4. VOLUME DETACH ISSUES
   → A mounted volume won't unmount
   → Fix: the underlying storage issue
```

### Force Delete (Last Resort)

```bash
# Only when you're sure the pod is actually gone / node is dead
kubectl delete pod <name> --grace-period=0 --force

# WARNING: this removes the K8s record without confirming the container
# stopped. For StatefulSets with storage, this risks data issues — be careful.
```

---

## 7. CreateContainerConfigError

**The container can't be created because a referenced ConfigMap or Secret is missing.**

### Diagnosis

```bash
kubectl describe pod <name>
# Events:
#   Error: configmap "app-config" not found
#   Error: secret "db-credentials" not found
```

### Fix

```bash
# Check what the pod references
kubectl describe pod <name> | grep -A5 "Environment\|Mounts"

# Check if the ConfigMap/Secret exists
kubectl get configmap
kubectl get secret

# If missing, create it
kubectl create configmap app-config --from-literal=LOG_LEVEL=info

# Or check you're in the right namespace — it might exist elsewhere
kubectl get configmap -A | grep app-config
```

---

## Service / Networking Issues

**Pods are Running and Ready, but you can't reach the service.**

### Diagnosis Flow

```bash
# 1. Does the Service exist and have endpoints?
kubectl get service <name>
kubectl get endpoints <name>
# If ENDPOINTS is empty → the Service's selector matches no ready pods

# 2. Do the Service's selector and pod labels match?
kubectl describe service <name> | grep Selector
kubectl get pods --show-labels
# The selector MUST match the pod labels

# 3. Are the target pods Ready?
kubectl get pods -l <selector>
# Not-ready pods aren't added to endpoints

# 4. Test from inside the cluster
kubectl run tmp --rm -it --image=busybox -- sh
# Then: wget -O- http://<service-name>:<port>
#       nslookup <service-name>
```

### Common Causes

| Symptom | Cause | Fix |
|---------|-------|-----|
| **Empty endpoints** | Selector doesn't match pod labels | Align selector with labels |
| **Empty endpoints** | No pods are Ready | Fix readiness (Section 5) |
| **DNS fails** | CoreDNS issue or wrong service name | Check `kube-dns`/CoreDNS |
| **Connection refused** | Wrong targetPort | Match targetPort to containerPort |
| **Times out** | NetworkPolicy blocking | Check NetworkPolicies |

**Key insight:** The most common service issue is a **label/selector mismatch** — the Service selects `app=web` but the pods are labeled `app=webapp`. Result: empty endpoints, nothing to route to.

---

## Node Issues

**Pods failing because of the node they're on.**

```bash
# Check node status
kubectl get nodes
# web-node-2   NotReady   ...     ← problem node

# Why is it NotReady?
kubectl describe node web-node-2
# Conditions: MemoryPressure, DiskPressure, PIDPressure, Ready

# Node pressure meanings:
#   MemoryPressure  → node low on memory (will evict pods)
#   DiskPressure    → node low on disk (will evict pods)
#   PIDPressure     → too many processes
```

**Node pressure → evictions:** When a node is under resource pressure, the kubelet evicts pods (BestEffort first, then Burstable, Guaranteed last — see [probes-resources.md](./probes-resources.md)).

---

## Interview Scenarios

### Scenario 1: "A pod is in CrashLoopBackOff. Debug it."

**Strong answer:**
```
"CrashLoopBackOff means the container starts, crashes, and restarts 
repeatedly. My flow:

1. kubectl describe pod — I check two things: 'Last State' for the 
   reason and exit code, and the Events at the bottom. Exit code 137 
   means OOMKilled; exit 1 is usually an app error.

2. kubectl logs <pod> --previous — this is the key step. The current 
   container just restarted so its logs are empty; --previous shows 
   the container that actually crashed, with the real stack trace.

Common causes I'd check: an app error on startup (bad config, missing 
dependency), a missing ConfigMap/Secret, or — a subtle one — a 
livenessProbe killing a slow-starting app before it finishes booting. 
For that last case, the fix is a startupProbe.

Once I see the actual error in --previous logs, the fix usually 
follows directly."
```

### Scenario 2: "A pod is stuck Pending. What's wrong?"

**Strong answer:**
```
"Pending means the scheduler can't place the pod on any node. I'd run 
kubectl describe pod and read the Events — they state the reason 
explicitly, like '0/3 nodes available: insufficient memory.'

The most common cause is resource requests: no node has enough free 
CPU or memory to satisfy the pod's requests (the scheduler uses 
requests, not limits). Other causes: a nodeSelector or affinity rule 
that no node matches, taints the pod doesn't tolerate, or an unbound 
PersistentVolumeClaim.

For the resource case, I'd check `kubectl top nodes` and the pod's 
requests. Fix is one of: lower overspecified requests, add nodes, or 
free capacity by scaling down other workloads."
```

### Scenario 3: "The Service exists but requests fail. Debug."

**Strong answer:**
```
"My first check is the endpoints: kubectl get endpoints <service>. If 
it's empty, the Service isn't routing to any pods, and that's the 
problem to solve.

The most common reason for empty endpoints is a label/selector 
mismatch — the Service selects app=web but the pods are labeled 
app=webapp. I'd compare `kubectl describe service` (the selector) 
against `kubectl get pods --show-labels`.

The second common reason is that pods aren't Ready — a failing 
readinessProbe keeps them out of the endpoints list. So I'd check 
pod readiness too.

If endpoints look fine, I'd test from inside the cluster with a 
temporary busybox pod — nslookup the service name and wget it — to 
separate DNS issues from connectivity, and check for NetworkPolicies 
that might be blocking traffic."
```

### Scenario 4: "Pod keeps getting OOMKilled. Walk me through it."

**Strong answer:**
```
"OOMKilled, exit code 137 — the container exceeded its memory limit 
and the kernel killed it. I confirm in kubectl describe (Last State: 
OOMKilled) and note the limit it hit.

The important question is WHY it hit the limit. I'd use kubectl top 
pod to watch memory over time:

- If usage is stable but just above the limit, the limit's too low — 
  the app genuinely needs more. I raise the limit.

- If usage grows continuously until it's killed, it's a memory leak. 
  Here the limit is actually protecting the node — raising it just 
  delays the crash. The real fix is in the application code.

So I don't reflexively bump the limit. I diagnose stable-vs-growing 
first, because a leak needs a code fix, not a bigger limit."
```

---

## Interview Tips

1. **Map each status to its cause** — CrashLoop (crashing), Pending (unschedulable), OOMKilled (memory), ImagePull (image)
2. **`logs --previous` for CrashLoopBackOff** — the single most important troubleshooting trick
3. **Exit 137 = OOMKilled** — know the key exit codes
4. **Pending = read the describe Events** — the scheduler tells you exactly why
5. **Empty endpoints = label/selector mismatch** — most common service bug
6. **OOMKilled: diagnose leak vs low limit** — don't just raise the limit
7. **Describe Events + logs are 90% of debugging** — always start there

---

## Common Pitfalls

1. **Reflexively raising memory limits on OOMKilled** — check for a leak first
2. **Using `logs` instead of `logs --previous`** on a crashing pod
3. **Not reading describe Events** — they usually state the exact problem
4. **Ignoring label/selector matching** for service issues
5. **Force-deleting Terminating pods carelessly** — risky for StatefulSets with storage
6. **Assuming "Running" means healthy** — check READY (readiness) too
