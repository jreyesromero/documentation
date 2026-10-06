# SRE Fundamentals - SLI, SLO, SLA & Reliability Concepts

The core vocabulary of Site Reliability Engineering. These concepts come up in almost every SRE interview and define how reliability is measured, committed to, and prioritized.

## The Three Key Metrics: SLI, SLO, SLA

These three terms are often confused. Here's the clear distinction:

| Term | Full Name | What It Is | Who It's For |
|------|-----------|-----------|--------------|
| **SLI** | Service Level **Indicator** | What you **measure** | Engineers |
| **SLO** | Service Level **Objective** | The **target** you aim for | Internal teams |
| **SLA** | Service Level **Agreement** | The **contract** with consequences | Customers/Legal |

### The Relationship

```
SLI (measurement)  →  SLO (internal target)  →  SLA (external contract)
   "99.95%"              "we want 99.9%"           "guarantee 99.5% or refund"
   
   What IS              What we WANT              What we PROMISE
```

**Key insight:** Your SLO should always be *stricter* than your SLA. You want internal alarms to fire **before** you breach a customer contract.

---

### SLI - Service Level Indicator

**What you measure.** A quantitative measure of some aspect of the service.

**Good SLIs:**
- Request latency (e.g., "95% of requests complete in < 200ms")
- Availability (e.g., "percentage of successful requests")
- Error rate (e.g., "percentage of requests returning 5xx")
- Throughput (e.g., "requests per second handled")

**Example:**
```
SLI: "The proportion of HTTP requests that return a 2xx/3xx status 
      in under 200ms, measured over a rolling 28-day window."
      
Current value: 99.95%
```

**Characteristics of a good SLI:**
- Directly reflects user experience
- Measurable and objective
- Expressed as a ratio (good events / total events)

---

### SLO - Service Level Objective

**The target.** The goal value (or range) for an SLI.

**Example:**
```
SLI:  Request success rate
SLO:  99.9% of requests succeed over 28 days
```

**Why SLOs matter:**
- They define "reliable enough" — 100% is the wrong target (impossible and expensive)
- They drive engineering decisions (do we ship features or fix reliability?)
- They set expectations across teams

**The "nines" of availability:**

| SLO | Downtime/year | Downtime/month | Downtime/week |
|-----|--------------|---------------|---------------|
| 90% (1 nine) | 36.5 days | 3 days | 16.8 hours |
| 99% (2 nines) | 3.65 days | 7.2 hours | 1.68 hours |
| 99.9% (3 nines) | 8.76 hours | **43.2 min** | 10.1 min |
| 99.95% | 4.38 hours | 21.6 min | 5 min |
| 99.99% (4 nines) | 52.6 min | 4.32 min | 1 min |
| 99.999% (5 nines) | 5.26 min | 25.9 sec | 6 sec |

**Memorize:** 99.9% = ~43 minutes of allowed downtime per month. This is the most commonly cited number.

---

### SLA - Service Level Agreement

**The contract.** A formal, often legal, agreement with customers that includes **consequences** for not meeting targets.

**Example:**
```
SLA: "We guarantee 99.5% monthly uptime. If we fall below this, 
      customers receive a 10% credit on their monthly bill."
```

**Key points:**
- SLAs have financial/legal consequences (refunds, credits, penalties)
- SLAs are typically **looser** than SLOs (buffer room)
- Not all services have SLAs (internal services often just have SLOs)

### Full Example (How They Fit Together)

```
SLI: "99.5% of requests complete in under 200ms"  ← what we measure
SLO: "We target 99.5%"                             ← our internal goal
SLA: "Customers can request a refund if < 99%"     ← our contract

Notice: SLO (99.5%) is stricter than SLA (99%).
That 0.5% gap is our safety buffer.
```

---

## Error Budget

One of the most important SRE concepts. The error budget is the amount of unreliability you're **allowed** to have.

### The Formula

```
Error Budget = 100% − SLO
```

**Example:**
```
SLO = 99.9%
Error Budget = 100% − 99.9% = 0.1%

Over a 30-day month:
0.1% of 30 days = 43.2 minutes of allowed downtime/errors
```

### Why Error Budgets Are Powerful

They turn reliability into a **resource you can spend**, which resolves the eternal tension between:
- **Development teams** who want to ship features fast (risky)
- **Operations teams** who want stability (conservative)

**The deal:**
- If you have error budget **remaining** → ship features, take risks, move fast
- If you've **burned** your error budget → freeze features, focus on reliability

### Error Budget in Practice

```
Scenario: Your SLO is 99.9% (43 min/month budget)

Week 1: A bad deploy caused 20 min of errors
Week 2: An incident caused 15 min of downtime
         → You've used 35 of 43 minutes (81% of budget)

Decision: Only 8 minutes of budget left this month.
          → Pause risky deployments
          → Prioritize reliability work
          → Be extra careful with changes
```

**Interview gold:** "If you've already burned your error budget, you must prioritize reliability over new features." This shows you understand the *purpose* of error budgets — they're a decision-making tool, not just a metric.

---

## Recovery & Detection Metrics

### MTTR - Mean Time To Recovery

**Average time to recover from a failure.** From the moment something breaks to when it's fixed.

```
MTTR = Total downtime / Number of incidents

Example: 
  4 incidents, total 2 hours downtime
  MTTR = 120 min / 4 = 30 minutes average recovery
```

**Lower MTTR = better.** Ways to reduce MTTR:
- Good monitoring (detect fast)
- Runbooks (know what to do)
- Automation (fix fast)
- Rollback capability (undo fast)

### MTTD - Mean Time To Detect

**Average time to *detect* a problem.** From when it breaks to when you know about it.

```
MTTD = Time from failure start → alert fired / acknowledged
```

**Why it matters:** You can't fix what you don't know is broken. High MTTD means problems fester before anyone notices.

### MTBF - Mean Time Between Failures

**Average time between incidents.** Higher is better (more stable).

```
MTBF = Total operational time / Number of failures
```

### The Timeline of an Incident

```
    Failure        Detection      Response       Recovery
      |               |              |              |
      |<--- MTTD ---->|              |              |
      |               |<- response ->|              |
      |<------------- MTTR --------------------->|
      |                                             |
   break            alert          start fix      fixed
```

---

## Other Essential Concepts

### Toil

**Repetitive, manual, automatable work that scales with service growth** but adds no lasting value.

**Characteristics of toil:**
- Manual (you do it by hand)
- Repetitive (same thing over and over)
- Automatable (a machine could do it)
- Reactive (interrupt-driven)
- No enduring value (doesn't improve the service)
- Scales linearly with growth (more users = more toil)

**Examples of toil:**
- Manually restarting a service every time it crashes
- Manually provisioning servers for each new customer
- Responding to the same alert the same way every time

**The SRE goal:** Keep toil below ~50% of time. Automate the rest so engineers can do engineering.

```
Bad:  "I SSH in and restart the service every time it hangs" (toil)
Good: "I wrote a health check that auto-restarts it + fixed the root cause" (engineering)
```

### Incident Management

The structured process for responding to outages:

```
1. Detect     → Monitoring/alerting catches the problem
2. Triage     → How bad? Who's affected? Severity level?
3. Respond    → Assign incident commander, communicate
4. Mitigate   → Stop the bleeding (rollback, failover, scale)
5. Resolve    → Full fix applied and verified
6. Postmortem → Learn from it (blameless)
```

### Postmortem (Blameless)

A written analysis after an incident. The **blameless** part is critical.

**Blameless means:** Focus on *what* went wrong (systems, processes), not *who* did it. People make mistakes in good faith; the system should have prevented or caught it.

**A good postmortem includes:**
- Timeline of events
- Root cause analysis
- Impact (users affected, duration, revenue)
- What went well / what went poorly
- Action items to prevent recurrence

```
❌ Blameful:  "John deployed bad code and took down prod."
✅ Blameless: "A code change with a null-pointer bug reached prod 
              because our CI didn't have a test for this case. 
              Action: add test coverage + staging canary."
```

### Alert Fatigue

When engineers receive **too many alerts** (especially false ones), they start ignoring them — including the real ones.

**Causes:**
- Alerting on causes instead of symptoms
- Thresholds too sensitive (CPU > 80% when that's normal)
- Non-actionable alerts (nothing to do about it)

**Fix:** Alert only on things that are (1) real impact and (2) actionable. See [monitoring-alerting.md](./monitoring-alerting.md).

---

## Interview Scenarios

### Scenario 1: "Explain SLI, SLO, and SLA"

**Strong answer:**
```
"They build on each other:

- SLI is what I MEASURE — a concrete metric like 'percentage of 
  requests under 200ms.' It's a number.

- SLO is the TARGET for that SLI — 'we want 99.9% of requests 
  under 200ms.' It's our internal goal.

- SLA is the CONTRACT with customers — 'if we drop below 99.5%, 
  you get a refund.' It has consequences.

The key relationship: SLO should be stricter than SLA, so internal 
alarms fire before we breach a customer promise. And the SLO 
defines our error budget — the unreliability we're allowed."
```

### Scenario 2: "What is an error budget and how do you use it?"

**Strong answer:**
```
"Error budget is 100% minus your SLO. If my SLO is 99.9%, I have 
a 0.1% error budget — about 43 minutes a month.

The power is in how it drives decisions. If I have budget left, 
the team can move fast and ship features — we can afford some risk. 
But if we've burned the budget through incidents, we freeze risky 
work and focus on reliability until we recover.

It resolves the classic dev-vs-ops tension by making reliability 
a measurable resource both sides agree on, instead of a subjective 
argument."
```

### Scenario 3: "How would you reduce MTTR?"

**Strong answer:**
```
"MTTR has several phases, and I'd attack each:

1. Detection (MTTD): Better monitoring and alerting so we know 
   fast — alert on symptoms users feel, not noisy metrics.

2. Diagnosis: Good dashboards and runbooks so the on-call engineer 
   can quickly see what's wrong and what to do.

3. Mitigation: Make rollback trivial and fast. Often the fastest 
   recovery is 'undo the last change,' not 'debug the root cause 
   live.'

4. Automation: For known failure modes, automate the fix 
   (auto-restart, auto-failover, auto-scale).

The principle: recover first, then diagnose. Stop the user impact, 
then do the postmortem."
```

### Scenario 4: "What's the difference between MTTR and MTTD?"

**Strong answer:**
```
"MTTD is Mean Time To DETECT — how long from when something breaks 
until we know about it. MTTR is Mean Time To RECOVER — the full 
time from break to fix, which includes detection.

So MTTD is a component of MTTR. If MTTD is high, your MTTR can 
never be good — you can't fix what you don't know is broken. 
That's why detection (monitoring/alerting) is the foundation 
of fast recovery."
```

---

## Interview Tips

1. **Know the SLI/SLO/SLA distinction cold** — it's the most common SRE question
2. **Memorize 99.9% = 43 min/month** — the canonical availability number
3. **Understand error budgets as a *decision tool*** — not just a metric, but how you choose features vs reliability
4. **Know MTTR includes MTTD** — detection is part of recovery
5. **"Blameless" is the key word for postmortems** — always mention it
6. **Toil = manual + repetitive + automatable** — and the 50% cap goal
7. **Connect concepts** — error budget comes from SLO, SLA is looser than SLO, etc.

---

## Common Pitfalls

1. **Confusing SLO and SLA** — SLO is internal target, SLA is external contract with consequences
2. **Saying 100% is the goal** — it's not; 100% is impossibly expensive, error budgets acknowledge this
3. **Forgetting SLA < SLO** — your contract should be looser than your internal target
4. **Treating error budget as just a number** — its value is as a decision-making framework
5. **Describing blameful postmortems** — always emphasize blameless culture
6. **Confusing MTTR and MTBF** — MTTR is recovery time, MTBF is time *between* failures
