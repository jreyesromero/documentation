# Monitoring & Alerting

How you observe systems and decide when to wake someone up. Good alerting is a defining skill of a mature SRE — it's the difference between catching problems early and drowning in noise.

## Monitoring vs Observability

| Term | Meaning |
|------|---------|
| **Monitoring** | Watching *known* metrics for *known* problems (CPU, memory, error rate) |
| **Observability** | Being able to *ask new questions* about system behavior (why is THIS request slow?) |

Monitoring tells you *that* something is wrong. Observability helps you understand *why*.

---

## The Three Pillars of Observability

```
┌──────────────┐   ┌──────────────┐   ┌──────────────┐
│   METRICS    │   │     LOGS     │   │    TRACES    │
│              │   │              │   │              │
│ Numbers over │   │ Discrete     │   │ Request path │
│ time         │   │ events       │   │ across       │
│              │   │              │   │ services     │
│ "CPU 80%"    │   │ "Error: ..." │   │ "A→B→C 200ms"│
└──────────────┘   └──────────────┘   └──────────────┘
```

| Pillar | What It Is | Example Tools |
|--------|-----------|---------------|
| **Metrics** | Numeric measurements over time (aggregatable) | Prometheus, Grafana, Datadog |
| **Logs** | Timestamped records of discrete events | ELK, Loki, Splunk |
| **Traces** | The path of a request across services | Jaeger, Zipkin, OpenTelemetry |

**When to use which:**
- **Metrics:** "Is something wrong?" (dashboards, alerts)
- **Logs:** "What exactly happened?" (debugging a specific error)
- **Traces:** "Where is the time going?" (latency across microservices)

---

## The Four Golden Signals

Google SRE's recommended metrics to monitor for any user-facing system. If you can only measure four things, measure these:

| Signal | Question | Example |
|--------|----------|---------|
| **Latency** | How long do requests take? | p99 response time < 200ms |
| **Traffic** | How much demand? | Requests per second |
| **Errors** | How many requests fail? | 5xx error rate < 0.1% |
| **Saturation** | How "full" is the system? | CPU/memory/disk utilization |

**Memorize these — "the four golden signals" is a very common interview phrase.**

```
Latency     → are responses fast?
Traffic     → how much load?
Errors      → how many failures?
Saturation  → how close to capacity?
```

**Latency nuance:** Separate *successful* request latency from *failed* request latency. A fast error is still an error, and slow errors can skew your "latency looks fine" picture.

---

## What Makes a Good Alert?

This is the heart of alerting maturity, and a frequent interview question.

### Bad Alert

```
❌ CPU > 80%
```

**Why it's bad:**
- CPU at 80% might be perfectly normal (efficient use of resources)
- It's a *cause*, not a *symptom* — users don't care about CPU, they care about the service working
- It's not necessarily actionable — what do you DO about 80% CPU?
- It will fire constantly, causing alert fatigue

### Good Alert

```
✅ Request latency p99 > 500ms for 5 minutes
✅ Error rate > 5% for 5 minutes
✅ Pod restart rate > 2/min
```

**Why these are good:**
- They measure **real user impact** (slow/failing requests)
- They're **symptoms** users actually feel, not internal causes
- They're **actionable** (something is clearly wrong, investigate)
- The **"for 5 minutes"** avoids flapping on momentary spikes

### The Rule

> **An alert should indicate real impact or real risk — not an abstract metric.**

```
Ask of every alert:
1. Does this represent actual or imminent user impact?
2. Is it actionable — is there something to do?
3. Does it need a human NOW, or could it wait / be automated?

If "no" to any → it shouldn't page someone.
```

### Symptom-Based vs Cause-Based Alerting

```
Cause-based:   "CPU is high"           → might not matter to users
Symptom-based: "Requests are slow"     → users are definitely affected

Alert on SYMPTOMS (what users feel), use CAUSES for diagnosis.
```

CPU, memory, disk are great for *dashboards and diagnosis* — but alert on the *symptom* (slow/failing requests), then use the cause metrics to figure out why.

---

## Alert Fatigue

When engineers get **too many alerts** (especially false or non-actionable ones), they start ignoring them — including the real emergencies.

```
100 alerts/day, 95 are noise
    ↓
Engineers learn to ignore alerts
    ↓
The 5 real ones get ignored too
    ↓
A real outage is missed
```

**Causes:**
- Alerting on causes instead of symptoms (CPU, memory noise)
- Thresholds too sensitive (fires on normal variation)
- Non-actionable alerts (nothing to do)
- No deduplication (one problem fires 50 alerts)

**Fixes:**
- Alert only on symptoms with real user impact
- Add duration conditions ("for 5 minutes")
- Make every alert actionable (link to a runbook)
- Group related alerts (one incident = one page)
- Regularly review and prune noisy alerts

---

## Alerting Severity Levels

Not everything needs to wake someone at 3 AM.

| Level | Meaning | Response |
|-------|---------|----------|
| **Page (critical)** | User-impacting, needs immediate action | Wake someone up now |
| **Ticket (warning)** | Needs attention, but not urgent | Handle during business hours |
| **Info/Log** | Record for context, no action needed | No notification |

**Key principle:** Only *page* for things that genuinely need a human *right now*. Everything else is a ticket or a log.

---

## Push vs Pull Monitoring

```
Pull (Prometheus model):   Monitoring system SCRAPES metrics from targets
                           "Prometheus asks each service: give me your metrics"

Push (StatsD model):       Services SEND metrics to the monitoring system
                           "Each service pushes its metrics to the collector"
```

**Prometheus (pull)** is the dominant model in cloud-native/Kubernetes environments. Services expose a `/metrics` endpoint; Prometheus scrapes it on an interval.

---

## Interview Scenarios

### Scenario 1: "What makes a good alert?"

**Strong answer:**
```
"A good alert represents real user impact and is actionable. The 
classic bad example is 'CPU > 80%' — that might be totally normal, 
it's a cause not a symptom, and there's nothing obvious to DO about 
it. It'll fire constantly and cause alert fatigue.

A good alert is symptom-based: 'p99 latency > 500ms for 5 minutes' 
or 'error rate > 5% for 5 minutes.' These measure what users 
actually experience, they're clearly actionable, and the duration 
clause stops them flapping on momentary spikes.

My rule: alert on symptoms users feel, and keep cause-metrics like 
CPU for dashboards and diagnosis. And only PAGE for things that 
genuinely need a human right now — everything else is a ticket."
```

### Scenario 2: "What are the four golden signals?"

**Strong answer:**
```
"Latency, Traffic, Errors, and Saturation — from the Google SRE book. 
If I could only monitor four things on a user-facing service, it's 
these:

- Latency: how long requests take (and I'd separate successful from 
  failed requests).
- Traffic: how much demand — requests per second.
- Errors: the rate of failing requests, like 5xx.
- Saturation: how full the system is — CPU, memory, disk, queue depth.

Together they give a complete picture of service health from the 
user's perspective. Latency and errors tell me if users are having 
a bad time; traffic and saturation tell me if I'm about to run out 
of capacity."
```

### Scenario 3: "Your team is suffering from alert fatigue. What do you do?"

**Strong answer:**
```
"Alert fatigue means too much noise, so people ignore alerts — 
including the real ones. I'd attack it systematically:

1. Audit the alerts: which fire most often, and how many are 
   actionable? Usually a handful of noisy alerts cause most pages.

2. Convert cause-based to symptom-based: replace 'CPU high' with 
   'requests slow.' Alert on what users feel.

3. Add duration conditions: 'for 5 minutes' kills alerts that 
   flap on momentary spikes.

4. Downgrade non-urgent alerts: if it doesn't need a human NOW, 
   it's a ticket, not a page.

5. Deduplicate: one incident shouldn't fire 50 alerts.

6. Make every remaining alert link to a runbook — if there's 
   nothing to do, it shouldn't page.

The goal: every page should be real, urgent, and actionable. If 
an alert doesn't meet that bar, it gets fixed or deleted."
```

### Scenario 4: "Metrics vs logs vs traces — when do you use each?"

**Strong answer:**
```
"They answer different questions:

Metrics are numbers over time — 'error rate is 2%.' I use them for 
dashboards and alerts: they tell me THAT something is wrong, cheaply 
and at scale.

Logs are discrete event records — 'request X failed with this stack 
trace.' I use them to understand WHAT exactly happened when I'm 
debugging a specific issue.

Traces follow a single request across services — 'this request spent 
200ms in service A, 50ms in B.' I use them to find WHERE time or 
failures happen in a distributed system.

Typical flow: a metric alert tells me errors are up, I check logs 
to see the specific error, and if it's a latency issue across 
microservices, I use traces to find the slow hop."
```

---

## Interview Tips

1. **"CPU > 80% is a bad alert"** — the canonical example; know WHY (cause not symptom, not actionable)
2. **Memorize the four golden signals** — Latency, Traffic, Errors, Saturation
3. **Symptom-based alerting** — alert on what users feel, use causes for diagnosis
4. **"Real impact AND actionable"** — the test for every alert
5. **Three pillars** — metrics (that), logs (what), traces (where)
6. **Alert fatigue** — too much noise means real alerts get ignored
7. **Only page for urgent + actionable** — everything else is a ticket

---

## Common Pitfalls

1. **Alerting on causes (CPU, memory)** — alert on symptoms (slow/failing requests)
2. **No duration clause** — alerts flap on momentary spikes without "for N minutes"
3. **Paging for non-urgent issues** — causes fatigue; use tickets for non-emergencies
4. **Confusing monitoring and observability** — monitoring = known problems, observability = new questions
5. **Forgetting to separate successful vs failed latency** — fast errors hide in blended latency
6. **One incident firing many alerts** — deduplicate and group
