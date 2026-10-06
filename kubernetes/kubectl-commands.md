# kubectl - The Debugging Toolkit

`kubectl` is how you interact with a Kubernetes cluster. For the hands-on troubleshooting round, these commands are your muscle memory — know the core debugging flow cold.

## Quick Reference

```bash
# Viewing resources
kubectl get pods                             # List pods in current namespace
kubectl get pods -n mynamespace              # List pods in specific namespace
kubectl get pods -A                          # All namespaces
kubectl get pods -o wide                     # More detail (node, IP)
kubectl get pods --watch                     # Live updates (-w)

# The debugging trio
kubectl describe pod <name>                  # Detailed state + events
kubectl logs <pod>                           # Container logs
kubectl get events                           # Recent cluster events

# Logs variations
kubectl logs <pod> --previous                # Logs from the PREVIOUS (crashed) container
kubectl logs <pod> -f                        # Follow (like tail -f)
kubectl logs <pod> -c <container>            # Specific container in multi-container pod

# Interactive
kubectl exec -it <pod> -- sh                 # Shell inside the pod
kubectl exec <pod> -- <command>              # Run one command

# Resource usage
kubectl top pod                              # CPU/memory per pod
kubectl top node                             # CPU/memory per node
```

---

## The Core Debugging Flow

When something's wrong, this is the sequence. **Memorize this order** — it's how you win the troubleshooting round:

```
1. kubectl get pods              → What's the STATUS? (CrashLoopBackOff? Pending? Running?)
                                   ↓
2. kubectl describe pod <name>   → WHY? (Events at the bottom tell the story)
                                   ↓
3. kubectl logs <name>           → What did the APP say? (application errors)
   kubectl logs <name> --previous  (if it restarted — logs from the dead container)
                                   ↓
4. kubectl get events            → What's happening cluster-wide?
                                   ↓
5. kubectl exec -it <name> -- sh → Get inside to inspect (if it's running)
```

**The mantra:** `get` → `describe` → `logs` → `events`. Status first, then why, then what the app said.

---

## kubectl get

Lists resources and their status.

```bash
# Pods
kubectl get pods
kubectl get pods -n production
kubectl get pods -A                          # all namespaces
kubectl get pods -o wide                     # + node and IP columns
kubectl get pods --show-labels               # + labels
kubectl get pods -l app=web                  # filter by label

# Other resources
kubectl get deployments
kubectl get services        (or: kubectl get svc)
kubectl get nodes
kubectl get ingress
kubectl get all                              # most common resource types

# Output formats
kubectl get pod <name> -o yaml               # full YAML definition
kubectl get pod <name> -o json               # JSON
```

### Reading `get pods` Output

```
NAME                   READY   STATUS             RESTARTS   AGE
web-7d4f8-x8k2         1/1     Running            0          5d
web-7d4f8-9j3m         0/1     CrashLoopBackOff   8          12m
api-5c9b2-m4n7         0/1     Pending            0          3m
db-0                   1/1     Running            0          20d
```

| Column | Meaning |
|--------|---------|
| **READY** | Containers ready / total. `0/1` = not ready |
| **STATUS** | Running, Pending, CrashLoopBackOff, Error, Completed, Terminating |
| **RESTARTS** | How many times it restarted. High number = problem |
| **AGE** | How long since created |

**Red flags:** `0/1 READY`, high `RESTARTS`, status anything other than `Running`/`Completed`.

---

## kubectl describe

**The single most useful debugging command.** Shows the full state of a resource, and — crucially — the **Events** at the bottom.

```bash
kubectl describe pod <name>
kubectl describe pod <name> -n mynamespace
kubectl describe node <node-name>
kubectl describe deployment <name>
```

### What to Look For in `describe pod`

```
Name:         web-7d4f8-9j3m
Status:       Running
Containers:
  web:
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       OOMKilled          ← THE SMOKING GUN
      Exit Code:    137
    Restart Count:  8
    Limits:
      memory:  256Mi                   ← the limit it exceeded
Events:                                ← ALWAYS READ THESE
  Type     Reason     Age   Message
  ----     ------     ----  -------
  Warning  BackOff    2m    Back-off restarting failed container
  Normal   Pulled     5m    Successfully pulled image
```

**The two gold mines:**
1. **Last State / Reason** — why the container last died (OOMKilled, Error, etc.)
2. **Events** (bottom) — the chronological story (scheduling, image pulls, failures)

**Interview habit:** Always say "I'd check the Events section of `kubectl describe`" — it's where the answer usually is.

---

## kubectl logs

Shows container stdout/stderr — the application's own output.

```bash
kubectl logs <pod>                           # current container logs
kubectl logs <pod> --previous                # PREVIOUS container (before last restart)
kubectl logs <pod> -f                        # follow (stream live)
kubectl logs <pod> --tail=50                 # last 50 lines
kubectl logs <pod> --since=1h                # last hour
kubectl logs <pod> -c <container>            # specific container (multi-container pod)
kubectl logs -l app=web                      # logs from all pods with label
```

### The `--previous` Flag (Critical)

```
Problem: Pod is in CrashLoopBackOff. It keeps dying and restarting.
         `kubectl logs <pod>` shows the NEW container (just started, no error yet).

Solution: kubectl logs <pod> --previous
         Shows the logs from the container that CRASHED — the actual error.
```

**This is a key troubleshooting insight:** for a crashing pod, the *current* logs are often empty or just-started. The error is in the *previous* (dead) container's logs. `--previous` is how you find why it crashed.

---

## kubectl get events

Shows recent cluster-wide events — scheduling decisions, image pulls, failures, evictions.

```bash
kubectl get events                           # all events, current namespace
kubectl get events -n production
kubectl get events --sort-by='.lastTimestamp'   # chronological
kubectl get events --field-selector type=Warning  # warnings only
kubectl get events -A                        # all namespaces
```

### What Events Tell You

```
LAST SEEN   TYPE      REASON              OBJECT          MESSAGE
2m          Warning   FailedScheduling    pod/api-5c9b2   0/3 nodes available: insufficient memory
5m          Warning   BackOff             pod/web-9j3m    Back-off restarting failed container
8m          Warning   Failed              pod/img-x2k9    Failed to pull image "myapp:tyop"
```

**Events reveal cluster-level problems** that `describe pod` on a single pod might not: scheduling failures, image pull issues, node problems, evictions.

---

## kubectl exec

Run commands inside a running container — for live inspection.

```bash
# Interactive shell
kubectl exec -it <pod> -- sh
kubectl exec -it <pod> -- bash               # if bash is available

# Single command
kubectl exec <pod> -- ls /app
kubectl exec <pod> -- cat /etc/config/app.conf
kubectl exec <pod> -- env                    # check environment variables
kubectl exec <pod> -- nslookup db-service    # test DNS from inside
kubectl exec <pod> -- curl localhost:8080/health   # test the app locally

# Specific container
kubectl exec -it <pod> -c <container> -- sh
```

**Common uses when debugging:**
- Check if config files / env vars are correct (`env`, `cat config`)
- Test network connectivity from inside (`curl`, `nslookup`, `nc`)
- Inspect the filesystem (`ls`, `df`)
- Verify the app responds locally (`curl localhost:PORT`)

**Note:** `exec` only works on *Running* containers. For a crashing pod, you can't exec in — use `logs --previous` and `describe` instead.

---

## kubectl top

Shows actual resource usage (requires metrics-server).

```bash
kubectl top pod                              # CPU/memory per pod
kubectl top pod -n production
kubectl top pod --sort-by=memory             # sort by memory
kubectl top node                             # per-node usage
```

```
NAME             CPU(cores)   MEMORY(bytes)
web-7d4f8-x8k2   5m           120Mi
api-5c9b2-m4n7   250m         480Mi            ← high memory, near a 512Mi limit?
```

**Use for:** Confirming a memory leak (growing over time), finding resource hogs, checking if a pod is near its limits.

---

## Other Useful Commands

```bash
# Rollout management
kubectl rollout status deployment/<name>     # watch a rollout
kubectl rollout undo deployment/<name>       # ROLLBACK to previous version
kubectl rollout history deployment/<name>    # see revision history

# Scaling
kubectl scale deployment/<name> --replicas=5

# Apply / delete
kubectl apply -f manifest.yaml               # declarative apply
kubectl delete pod <name>                     # delete (Deployment will recreate it)

# Context / namespace
kubectl config current-context               # which cluster am I on?
kubectl config get-contexts                  # list clusters
kubectl config set-context --current --namespace=prod   # set default namespace

# Explain (built-in docs)
kubectl explain pod.spec.containers          # field documentation
```

### Rollback — The Fast Recovery

```bash
# A bad deploy broke prod. Recover instantly:
kubectl rollout undo deployment/web

# Verify
kubectl rollout status deployment/web
kubectl get pods
```

**Interview point:** The fastest incident recovery is often `kubectl rollout undo` — revert the deployment, *then* debug. Recovery first, root cause second.

---

## Interview Scenarios

### Scenario 1: "A pod is crashing. Walk me through your debugging."

**Strong answer:**
```
"My flow is always get → describe → logs → events:

1. kubectl get pods — see the status. Say it shows CrashLoopBackOff 
   with 8 restarts.

2. kubectl describe pod <name> — I go straight to two things: the 
   'Last State' (why it last died — maybe OOMKilled, exit code 137) 
   and the Events at the bottom (the chronological story).

3. kubectl logs <name> --previous — this is key. Current logs show 
   the fresh container with no error yet. --previous shows the 
   container that actually CRASHED, so I see the real error.

4. kubectl get events — if describe didn't explain it, cluster events 
   might (scheduling, image, node issues).

5. If it's running long enough, kubectl exec -it to get inside and 
   check config, env vars, connectivity.

The describe Events and logs --previous are usually where I find the 
answer."
```

### Scenario 2: "Why would you use `logs --previous`?"

**Strong answer:**
```
"When a pod is in CrashLoopBackOff, it keeps dying and restarting. If 
I run plain `kubectl logs`, I'm looking at the CURRENT container — 
which just started and hasn't hit the error yet, so the logs are 
often empty or incomplete.

`--previous` gives me the logs from the container instance that 
actually crashed — the one before the latest restart. That's where 
the stack trace or error message lives. It's the difference between 
seeing nothing and seeing the actual cause of the crash."
```

---

## Interview Tips

1. **Memorize the flow:** get → describe → logs → events
2. **`describe` Events section** — always mention it; the answer is usually there
3. **`logs --previous`** — the key to debugging CrashLoopBackOff
4. **`exec` only works on Running pods** — use logs/describe for crashing ones
5. **`rollout undo`** — the fast recovery for a bad deploy
6. **`top`** — confirms memory leaks and resource hogs
7. **`-A` and `-n`** — know how to target namespaces

---

## Common Pitfalls

1. **Using plain `logs` on a crashing pod** — use `--previous` to see the crash
2. **Forgetting to check Events** — `describe` Events is the most underused gold mine
3. **Trying to `exec` into a crashing pod** — it's not running; use logs/describe
4. **Not specifying namespace** — resources may be in a different namespace (`-n` or `-A`)
5. **Debugging live instead of rolling back** — `rollout undo` first in an incident
