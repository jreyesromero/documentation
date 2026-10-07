# Kubernetes Guide

Welcome to the Kubernetes reference for SRE, DevOps, and Platform Engineers. This section covers both the **concepts** (for conceptual interview rounds) and **hands-on troubleshooting** (for practical debugging rounds) — the two ways Kubernetes shows up in SRE interviews.

## Table of Contents

### Concepts & Objects

- [Core Concepts](./concepts.md) - The Building Blocks
  - Pod, ReplicaSet, Deployment (the hierarchy)
  - Service (L4) vs Ingress (L7)
  - ConfigMap vs Secret (and why Secrets aren't encrypted)
  - Namespace, DaemonSet, StatefulSet
  - Workload types compared

- [Probes & Resources](./probes-resources.md) - Health & Scheduling
  - Liveness vs Readiness vs Startup probes (classic interview question)
  - Requests vs Limits (scheduling vs enforcement)
  - CPU throttling vs Memory OOMKill
  - QoS classes and eviction priority

### Hands-On Debugging

- [kubectl Commands](./kubectl-commands.md) - The Debugging Toolkit
  - The core flow: get → describe → logs → events
  - `logs --previous` for crashing pods
  - exec, top, rollout undo
  - Reading `get pods` and `describe` output

- [Troubleshooting](./troubleshooting.md) - Failure States & Fixes
  - CrashLoopBackOff, ImagePullBackOff, OOMKilled, Pending
  - Running-but-not-Ready, stuck Terminating
  - Service/networking issues (label-selector mismatch)
  - Node pressure and evictions

---

## Two Interview Rounds, One Section

Kubernetes typically appears in two places in an SRE interview:

```
Round 1 (conceptual):  "What's the difference between a Service and Ingress?"
                       "When would you use a StatefulSet?"
                       → See concepts.md + probes-resources.md

Round 2 (hands-on):    "This pod is in CrashLoopBackOff. Debug it."
                       "The service isn't reachable. What do you check?"
                       → See kubectl-commands.md + troubleshooting.md
```

---

## Quick Reference by Problem

### Pod won't start / keeps crashing
- CrashLoopBackOff → [Troubleshooting §1](./troubleshooting.md#1-crashloopbackoff)
- ImagePullBackOff → [Troubleshooting §2](./troubleshooting.md#2-imagepullbackoff--errimagepull)
- CreateContainerConfigError → [Troubleshooting §7](./troubleshooting.md#7-createcontainerconfigerror)

### Pod won't schedule / gets killed
- Pending → [Troubleshooting §3](./troubleshooting.md#3-pending)
- OOMKilled → [Troubleshooting §4](./troubleshooting.md#4-oomkilled)

### Pod runs but doesn't work
- Running but not Ready → [Troubleshooting §5](./troubleshooting.md#5-running-but-not-ready)
- Stuck Terminating → [Troubleshooting §6](./troubleshooting.md#6-stuck-terminating)

### Can't reach the service
- Service/networking → [Troubleshooting: Service Issues](./troubleshooting.md#service--networking-issues)

### Conceptual questions
- Pod/ReplicaSet/Deployment hierarchy → [Concepts](./concepts.md#deployment)
- Service vs Ingress → [Concepts](./concepts.md#ingress)
- Liveness vs Readiness → [Probes & Resources](./probes-resources.md#the-critical-distinction-liveness-vs-readiness)
- Requests vs Limits → [Probes & Resources](./probes-resources.md#resource-requests--limits)

---

## The Debugging Flow (Memorize This)

```
kubectl get pods              → What's the STATUS?
        ↓
kubectl describe pod <name>   → WHY? (Last State + Events)
        ↓
kubectl logs <name> --previous → What did the crashed app say?
        ↓
kubectl get events            → Cluster-level issues?
        ↓
kubectl exec -it <name> -- sh → Inspect from inside (if Running)
```

**Mantra:** `get → describe → logs → events`. Status first, then why, then what the app said.

---

## Interview Focus

**Must-know concepts:**
1. **Pod → ReplicaSet → Deployment** hierarchy
2. **Service (L4) vs Ingress (L7)**
3. **Liveness vs Readiness probes** — liveness restarts, readiness removes from traffic
4. **Requests vs Limits** — requests schedule, limits enforce
5. **Secrets are base64, not encrypted**
6. **StatefulSet for stateful apps** (databases)

**Must-know troubleshooting:**
1. **CrashLoopBackOff** → `logs --previous` to find the crash
2. **OOMKilled (137)** → diagnose leak vs low limit
3. **Pending** → read describe Events (usually insufficient resources)
4. **Service unreachable** → check endpoints / label-selector match
5. **The flow** → get → describe → logs → events

**Key numbers/facts:**
- Exit 137 = OOMKilled, Exit 143 = SIGTERM
- Memory over limit = killed; CPU over limit = throttled
- Scheduler uses *requests* (not limits) for placement

---

## Coverage Summary

| File | Focus | Interview Scenarios |
|------|-------|---------------------|
| concepts.md | Core objects & relationships | 4 |
| probes-resources.md | Health checks & resource management | 4 |
| kubectl-commands.md | The debugging toolkit | 2 |
| troubleshooting.md | Failure states & fixes | 4 |

---

**Note:** This section serves both the conceptual round (Round 1) and the hands-on troubleshooting round (Round 2). The concepts and probes files build the mental model; the kubectl and troubleshooting files build the debugging muscle memory.

**Key Principle:** Kubernetes troubleshooting is systematic, not magic. Almost every problem is diagnosed by the same flow — `get` to see status, `describe` to read the Events, `logs --previous` to see the crash. Master that flow and most failures become straightforward.
