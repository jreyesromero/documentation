# DevOps & SRE Fundamentals

Welcome to the DevOps/SRE fundamentals guide for SRE, DevOps, and Platform Engineers. This section covers the core concepts, vocabulary, and principles that define modern reliability engineering — the theory that complements the hands-on Linux and Networking skills.

## Table of Contents

### Reliability & Measurement

- [SRE Fundamentals](./sre-fundamentals.md) - SLI, SLO, SLA & Reliability Concepts
  - SLI vs SLO vs SLA (the most common SRE interview question)
  - Error budgets and the "nines" of availability
  - MTTR, MTTD, MTBF — recovery and detection metrics
  - Toil, incident management, blameless postmortems, alert fatigue

### Delivery & Automation

- [CI/CD](./cicd.md) - Continuous Integration & Delivery
  - CI vs Continuous Delivery vs Continuous Deployment
  - Artifacts and "build once, deploy many"
  - Immutable deployments
  - Pipeline stages and tooling

- [Deployment Strategies](./deployment-strategies.md) - Safe Rollout Patterns
  - Recreate, Rolling, Blue-Green, Canary
  - Blast radius and rollback speed trade-offs
  - The database compatibility problem (expand/contract)

- [Infrastructure as Code](./iac.md) - IaC Principles
  - Declarative vs Imperative
  - Idempotency, Drift, State, State locking
  - Terraform (provisioning) vs Ansible (configuration)

### Observability

- [Monitoring & Alerting](./monitoring-alerting.md) - Observability & Good Alerts
  - Metrics, logs, traces (the three pillars)
  - The four golden signals
  - What makes a good alert (symptom-based vs cause-based)
  - Alert fatigue and severity levels

---

## Quick Reference by Topic

### "Explain the reliability metrics"
- SLI / SLO / SLA distinction → [SRE Fundamentals](./sre-fundamentals.md)
- Error budget math (99.9% = 43 min/month) → [SRE Fundamentals](./sre-fundamentals.md)
- MTTR vs MTTD → [SRE Fundamentals](./sre-fundamentals.md)

### "Deployment and delivery"
- CI vs CD vs Continuous Deployment → [CI/CD](./cicd.md)
- Blue-Green vs Canary → [Deployment Strategies](./deployment-strategies.md)
- How to roll back safely → [Deployment Strategies](./deployment-strategies.md)

### "Infrastructure concepts"
- Declarative vs Imperative → [IaC](./iac.md)
- What is configuration drift → [IaC](./iac.md)
- Why Terraform needs state → [IaC](./iac.md)

### "Monitoring and alerting"
- Four golden signals → [Monitoring & Alerting](./monitoring-alerting.md)
- What makes a good alert → [Monitoring & Alerting](./monitoring-alerting.md)
- Metrics vs logs vs traces → [Monitoring & Alerting](./monitoring-alerting.md)

---

## Interview Focus

**Must-know concepts (highest frequency in interviews):**

1. **SLI / SLO / SLA** — the distinction, and that SLO is stricter than SLA
2. **Error budget** — 100% − SLO, and how it drives feature-vs-reliability decisions
3. **The four golden signals** — Latency, Traffic, Errors, Saturation
4. **Good vs bad alerts** — "CPU > 80%" (bad) vs "p99 latency > 500ms for 5 min" (good)
5. **Deployment strategies** — especially Blue-Green vs Canary trade-offs
6. **Declarative vs Imperative IaC** — end-state vs steps, and idempotency

**Key numbers to memorize:**
- 99.9% availability = ~43 minutes downtime/month
- Toil should stay below ~50% of an SRE's time
- Alert duration clauses: typically "for 5 minutes"

---

## How These Concepts Connect

```
SLO (target)  →  Error Budget (100% − SLO)  →  drives deployment risk decisions
                                                       ↓
Monitoring (golden signals)  →  Alerts (symptom-based)  →  Incident  →  MTTR
                                                                           ↓
                                                              Blameless Postmortem
                                                                           ↓
                                                              Action items (reduce toil,
                                                              fix root cause via IaC)
```

The concepts aren't isolated — SLOs define error budgets, which gate deployments; monitoring drives alerts, which trigger incident response, which produces postmortems, which drive automation. Understanding these *connections* is what separates a strong candidate from someone who just memorized definitions.

---

## Coverage Summary

| Topic | Key Concepts | Interview Scenarios |
|-------|-------------|---------------------|
| SRE Fundamentals | SLI/SLO/SLA, error budget, MTTR/MTTD, toil | 4 |
| CI/CD | CI/CD/CDeploy, artifacts, immutability | 3 |
| Deployment Strategies | Recreate/Rolling/Blue-Green/Canary | 3 |
| IaC | Declarative/imperative, idempotency, drift, state | 3 |
| Monitoring & Alerting | Golden signals, good alerts, 3 pillars | 4 |

---

**Note:** This section is concept-focused rather than command-focused — it's the vocabulary and principles of reliability engineering. Pair it with the hands-on [Linux](../linux/linux.md) and [Networking](../networking/networking.md) sections for complete coverage of the first interview round (Linux + Networking + DevOps basics).

**Key Principle:** SRE is about making reliability a measurable, engineered property of systems — not hoping things stay up. Every concept here serves that goal: measure it (SLIs), target it (SLOs), budget it (error budgets), protect it (deployment strategies), observe it (monitoring), and improve it (blameless postmortems).
