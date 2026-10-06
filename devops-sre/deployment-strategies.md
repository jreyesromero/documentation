# Deployment Strategies

How you roll out new versions determines your blast radius when something goes wrong. Knowing the trade-offs between strategies is a common SRE interview topic.

## Overview

| Strategy | How It Works | Risk | Rollback Speed | Cost |
|----------|-------------|------|---------------|------|
| **Recreate** | Stop old, start new | High (downtime) | Slow | Low |
| **Rolling** | Replace instances gradually | Medium | Medium | Low |
| **Blue-Green** | Two full environments, switch traffic | Low | Instant | High (2x) |
| **Canary** | Small % first, then expand | Lowest | Fast | Low-Medium |

---

## Recreate (Basic)

Stop all old instances, then start all new ones.

```
[v1][v1][v1]  →  [  ][  ][  ]  →  [v2][v2][v2]
 running          DOWNTIME         running
```

**Pros:** Simple, no version mismatch (never v1 and v2 at once)
**Cons:** Downtime during the switch

**Use when:** Downtime is acceptable (internal tools, batch jobs), or when v1 and v2 absolutely cannot coexist.

---

## Rolling Deployment

**Replace instances gradually**, a few at a time, until all are updated.

```
Step 1:  [v1][v1][v1][v1]
Step 2:  [v2][v1][v1][v1]   ← replace one
Step 3:  [v2][v2][v1][v1]   ← replace another
Step 4:  [v2][v2][v2][v1]
Step 5:  [v2][v2][v2][v2]   ← done
```

**Pros:**
- No downtime (always some instances serving)
- No extra infrastructure cost (reuse same capacity)
- The default in Kubernetes

**Cons:**
- Slow (gradual)
- Both versions run simultaneously (must be compatible)
- Rollback is also gradual (not instant)

**Key requirement:** v1 and v2 must be able to coexist — same database schema, compatible APIs. If a request can hit either version, both must handle it correctly.

**Use when:** You want zero-downtime with minimal cost (most common default).

---

## Blue-Green Deployment

**Maintain two complete, identical environments.** Only one serves live traffic at a time.

```
         ┌─────────────┐
         │Load Balancer│
         └──────┬──────┘
                │ (all traffic)
         ┌──────▼──────┐        ┌─────────────┐
         │   BLUE      │        │   GREEN     │
         │   (v1)      │        │   (v2)      │
         │   LIVE      │        │   idle/test │
         └─────────────┘        └─────────────┘

After switch:
         ┌─────────────┐
         │Load Balancer│
         └──────┬──────┘
                │ (all traffic)
         ┌─────────────┐        ┌──────▼──────┐
         │   BLUE      │        │   GREEN     │
         │   (v1)      │        │   (v2)      │
         │   idle      │        │   LIVE      │
         └─────────────┘        └─────────────┘
```

**The process:**
1. Blue (v1) is live, serving all traffic
2. Deploy v2 to Green (idle environment)
3. Test Green thoroughly (it's not serving users yet)
4. Switch the load balancer: all traffic → Green
5. Green is now live; Blue is kept idle as instant rollback

**Pros:**
- **Instant switch** (just flip the load balancer)
- **Instant rollback** (flip back to Blue)
- Test the new version in a production-identical environment before going live
- Zero downtime

**Cons:**
- **Expensive** — you need double the infrastructure (two full environments)
- Database migrations are tricky (both environments may share one DB)

**Use when:** You need instant rollback and can afford the 2x infrastructure cost. Common for critical services.

---

## Canary Deployment

**Release to a small percentage of users first.** If it's healthy, gradually expand.

```
Step 1:  95% traffic → [v1][v1][v1][v1]
          5% traffic → [v2]                ← canary (small exposure)

Step 2:  75% traffic → [v1][v1][v1]
         25% traffic → [v2][v2]            ← looking good, expand

Step 3:   0% traffic → 
         100% traffic → [v2][v2][v2][v2]   ← full rollout
```

**The name:** From "canary in a coal mine" — a small, early warning system. The canary (small % of traffic) detects danger before it affects everyone.

**The process:**
1. Deploy v2 alongside v1
2. Route a small % of traffic (e.g., 5%) to v2
3. **Monitor key metrics** (error rate, latency) on the canary
4. If healthy → increase traffic gradually (25%, 50%, 100%)
5. If unhealthy → route traffic back to v1, investigate

**Pros:**
- **Smallest blast radius** — only a few users affected if v2 is bad
- Real production traffic tests v2 (not synthetic)
- Data-driven rollout (metrics decide whether to proceed)

**Cons:**
- More complex (traffic splitting, monitoring automation)
- Slower full rollout
- Both versions run simultaneously (compatibility required)

**Use when:** You want to minimize risk and have good monitoring to evaluate the canary. Ideal for high-traffic services where 5% is still a meaningful sample.

---

## Rollback

**Quickly reverting to the previous working version** when a deployment goes wrong.

**Rollback speed by strategy:**
```
Blue-Green:  INSTANT      (flip load balancer back to Blue)
Canary:      FAST         (route traffic back to v1, which is still running)
Rolling:     MEDIUM       (roll instances back to v1, gradual)
Recreate:    SLOW         (stop v2, start v1 — downtime again)
```

**Key principle:** The best deployments make rollback trivial. Often the fastest incident recovery is "roll back the last change," not "debug the new version live."

**What makes rollback reliable:**
- Immutable artifacts (previous version is still available as an image)
- Backward-compatible database changes (new schema works with old code)
- Automated rollback triggers (metrics degrade → auto-revert)

---

## The Database Problem (Advanced)

All strategies where v1 and v2 coexist (Rolling, Canary, sometimes Blue-Green) share a challenge: **the database schema must work with both versions.**

```
Problem: v2 adds a new NOT NULL column.
         v1 doesn't know about it → v1 writes fail.

Solution: Expand/contract pattern (backward-compatible migrations)
  1. EXPAND:   Add column as nullable (both v1 and v2 work)
  2. MIGRATE:  Deploy v2 which uses the column
  3. CONTRACT: Once v1 is gone, make column NOT NULL
```

**Interview insight:** Mentioning that deployments and database migrations must be decoupled (backward-compatible schema changes) shows senior-level understanding.

---

## Interview Scenarios

### Scenario 1: "Compare Blue-Green and Canary deployments"

**Strong answer:**
```
"Both reduce deployment risk, but differently:

Blue-Green has two full environments. You deploy to the idle one, 
test it, then switch ALL traffic at once. The win is instant 
rollback — flip the load balancer back. The cost is 2x 
infrastructure, and the switch is all-or-nothing.

Canary routes a SMALL percentage of real traffic to the new 
version first. You monitor metrics on that slice, then gradually 
expand. The win is the smallest blast radius — if v2 is bad, only 
5% of users are affected, and you learn from real traffic. The 
cost is complexity: you need traffic splitting and good monitoring.

I'd pick Blue-Green when instant rollback is critical and I can 
afford the cost. I'd pick Canary for high-traffic services where 
I want to validate against real users with minimal exposure."
```

### Scenario 2: "You deploy and error rates spike. What happens next?"

**Strong answer:**
```
"First priority: stop the user impact — roll back, don't debug live.

How fast depends on the strategy:
- If Blue-Green: flip the load balancer back to Blue. Instant.
- If Canary: route the canary traffic back to v1. The old version 
  is still running, so it's fast.
- If Rolling: roll the instances back to v1.

Ideally this is automated — if the canary's error rate crosses a 
threshold, the system auto-reverts without waiting for a human.

Once users are safe, THEN I diagnose: pull the logs, reproduce in 
staging, write the postmortem. Recovery first, root cause second."
```

### Scenario 3: "Why is 100% uptime deployment hard?"

**Strong answer:**
```
"The challenge is that during a deploy, you have two versions in 
play, and they often share state — especially the database.

If v2 changes the schema in a way v1 can't handle, then while 
both are running (rolling/canary), one of them breaks. The 
solution is backward-compatible migrations: expand the schema 
first so both versions work, deploy the code, then contract later.

So zero-downtime isn't just about the deployment strategy — it's 
about making every change backward-compatible so old and new can 
coexist during the transition."
```

---

## Interview Tips

1. **Know all four strategies** and their blast-radius/cost trade-offs
2. **Blue-Green = instant rollback, 2x cost** — the defining characteristics
3. **Canary = smallest blast radius** — "canary in a coal mine"
4. **Rolling = Kubernetes default** — zero downtime, no extra cost, gradual
5. **"Recovery first, diagnose second"** — roll back before debugging
6. **Mention database compatibility** — expand/contract pattern shows depth
7. **Connect to error budgets** — good deployment strategies protect your budget

---

## Common Pitfalls

1. **Forgetting v1/v2 coexistence** — rolling and canary require compatible versions
2. **Ignoring the database** — schema changes can break the "both versions run" assumption
3. **Confusing Blue-Green and Canary** — Blue-Green switches all at once; Canary is gradual %
4. **Thinking rollback is always instant** — only Blue-Green is truly instant
5. **Debugging live instead of rolling back** — stop user impact first
