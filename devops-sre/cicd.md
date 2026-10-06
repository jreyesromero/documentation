# CI/CD - Continuous Integration & Delivery

The pipeline that takes code from a developer's machine to production safely and repeatedly. Core DevOps knowledge for SRE interviews.

## The Three "C"s - Often Confused

| Term | What Happens | Human Involved? |
|------|-------------|-----------------|
| **Continuous Integration (CI)** | Code merged frequently, auto-tested | Yes (merge decision) |
| **Continuous Delivery (CD)** | Code is *always ready* to deploy | Yes (deploy button) |
| **Continuous Deployment** | Code *auto-deploys* to prod | No (fully automated) |

### The Key Distinction

```
Continuous Integration:   commit → build → test        (stops at "tested artifact")
Continuous Delivery:      commit → build → test → [READY TO DEPLOY]  (human clicks deploy)
Continuous Deployment:    commit → build → test → DEPLOY  (no human, automatic)
```

**The subtle one:** Continuous *Delivery* vs Continuous *Deployment* both abbreviate to "CD." The difference is the final step:
- **Delivery** = ready to deploy, but a human decides *when*
- **Deployment** = deploys automatically with no human gate

---

## Continuous Integration (CI)

**Developers integrate code frequently** (multiple times a day), and each integration is **automatically verified** by building and running tests.

### What CI Solves

**Before CI (integration hell):**
```
Developers work in isolation for weeks
    ↓
Everyone merges at the end
    ↓
Massive conflicts, broken builds, "works on my machine"
    ↓
Days of painful integration
```

**With CI:**
```
Developers merge small changes frequently (daily)
    ↓
Each merge auto-builds and auto-tests
    ↓
Problems caught immediately, while the change is small
    ↓
Integration is continuous and painless
```

### A Typical CI Pipeline

```
1. Developer pushes code to branch
2. CI system detects the push (webhook)
3. Checkout code
4. Build / compile
5. Run unit tests
6. Run integration tests
7. Run linters / static analysis
8. Report results (pass/fail)
   → If pass: ready to merge
   → If fail: block merge, notify developer
```

### CI Best Practices

- **Commit frequently** (small changes are easier to test and debug)
- **Keep builds fast** (< 10 min ideally — slow builds discourage frequent commits)
- **Fix broken builds immediately** (a red build blocks everyone)
- **Test automation is mandatory** (manual testing doesn't scale)

---

## Continuous Delivery (CD)

**Every change that passes CI is automatically prepared for release.** The code is *always* in a deployable state, but a human decides when to actually push the button.

```
CI passes → artifact built → deployed to staging → [waiting for human approval] → production
```

**Why have a human gate?**
- Business timing (don't deploy during a big sale)
- Compliance (change approval requirements)
- Confidence (manual sign-off for high-risk changes)

---

## Continuous Deployment

**Every change that passes all automated checks goes straight to production** with no human intervention.

```
CI passes → artifact built → staging → automated checks pass → PRODUCTION (automatic)
```

**Requirements for Continuous Deployment:**
- Excellent test coverage (tests are your only safety net)
- Strong monitoring (catch problems fast)
- Easy rollback (undo quickly if something slips through)
- Feature flags (deploy code without exposing features)

**Trade-off:** Fastest delivery, but requires the most mature testing and monitoring. A bug that passes tests goes straight to users.

---

## Key Concepts

### Artifact

**The packaged, deployable output of a build.** It's what you actually deploy.

**Examples:**
- A compiled binary (Go, Rust, C++)
- A JAR/WAR file (Java)
- A Docker image (containerized apps)
- A tarball of built assets
- A Python wheel

```
Source code  →  [BUILD]  →  Artifact  →  [DEPLOY]  →  Running service
  (.go files)              (Docker image)            (container in prod)
```

**Key principle: build once, deploy many.** Build the artifact a single time, then promote that *same* artifact through environments (dev → staging → prod). Don't rebuild per environment — that introduces inconsistency.

### Immutable Deployment

**You don't modify what's deployed — you replace it entirely.**

```
❌ Mutable:   SSH into server, update files in place, restart
              (server state drifts, hard to reproduce, "works on my machine")

✅ Immutable: Build new artifact/image, deploy fresh instances, 
              destroy old ones
              (every deploy is clean, reproducible, rollback = redeploy old image)
```

**Benefits:**
- Reproducible (same image = same behavior everywhere)
- Easy rollback (just redeploy the previous image)
- No configuration drift (nothing is modified in place)
- Matches container/cloud patterns

### Pipeline Stages (Typical)

```
┌─────────┐   ┌───────┐   ┌──────┐   ┌─────────┐   ┌──────┐   ┌──────┐
│ Source  │ → │ Build │ → │ Test │ → │ Staging │ → │ Prod │ → │Verify│
└─────────┘   └───────┘   └──────┘   └─────────┘   └──────┘   └──────┘
  git push    compile +    unit +      deploy to    deploy to   health
              package      integration  staging env  prod        checks
```

---

## CI/CD Tools (Know the Names)

| Category | Tools |
|----------|-------|
| **CI/CD Platforms** | Jenkins, GitHub Actions, GitLab CI, CircleCI, Travis CI, Argo CD |
| **Build** | Make, Maven, Gradle, Bazel, Docker |
| **Artifact Registries** | Docker Hub, Amazon ECR, Artifactory, Nexus, GitHub Packages |
| **Deployment** | Argo CD, Flux, Spinnaker, Helm, Kubernetes |

**You don't need deep expertise in all — know what each category does and name a couple of examples.**

---

## Real-World Scenarios

### Scenario 1: "Walk me through your CI/CD pipeline"

**Strong structure:**
```
"A typical flow I'd describe:

1. SOURCE: Developer opens a PR. A webhook triggers the pipeline.

2. CI: We checkout the code, build it, and run the test suite — 
   unit tests, integration tests, linters. If anything fails, 
   the PR is blocked.

3. ARTIFACT: On merge to main, we build a Docker image once and 
   tag it (e.g., with the git SHA). This same image is promoted 
   through all environments — build once, deploy many.

4. STAGING: The image deploys automatically to staging. We run 
   smoke tests and maybe some end-to-end tests against it.

5. PRODUCTION: Depending on maturity — either a human approves 
   (Continuous Delivery) or it auto-deploys (Continuous Deployment). 
   We use a progressive strategy like canary to limit blast radius.

6. VERIFY: Post-deploy health checks and monitoring confirm the 
   release is healthy. If metrics degrade, we auto-rollback."
```

### Scenario 2: "What's the difference between Continuous Delivery and Continuous Deployment?"

**Strong answer:**
```
"Both take code through build and test automatically. The 
difference is the final step to production:

- Continuous DELIVERY: the artifact is always *ready* to deploy, 
  but a human clicks the button. You get deployment on-demand, 
  with a manual gate for timing or compliance.

- Continuous DEPLOYMENT: there's no human gate — if it passes all 
  automated checks, it goes straight to production.

Deployment is 'delivery minus the human.' It requires much more 
mature testing and monitoring, because your automated checks are 
the only thing between a commit and your users."
```

### Scenario 3: "A deploy broke production. How does CI/CD help you recover?"

**Strong answer:**
```
"Several ways CI/CD makes recovery fast:

1. Immutable artifacts: because we built an image and kept the 
   previous one, rollback is just redeploying the last-known-good 
   image — fast and reliable.

2. Build once, deploy many: the artifact that's in prod is the 
   exact one we tested in staging, so I can trust the previous 
   version will work.

3. Pipeline automation: I don't manually rebuild — I re-run the 
   deploy step pointing at the previous artifact.

The key enabler is immutability: I'm not trying to 'un-edit' 
files on servers, I'm swapping back to a known-good package."
```

---

## Interview Tips

1. **Nail the three definitions** — CI, Continuous Delivery, Continuous Deployment, and the human-gate distinction
2. **"Build once, deploy many"** — a principle that signals maturity
3. **Immutable deployment** — know why it beats modifying servers in place
4. **Artifact = deployable package** — Docker image is the modern canonical example
5. **Name a few tools per category** — Jenkins, GitHub Actions, Argo CD, etc.
6. **Connect to reliability** — fast rollback, limited blast radius, reproducibility

---

## Common Pitfalls

1. **Confusing Continuous Delivery and Deployment** — the only difference is the human gate before prod
2. **Rebuilding per environment** — breaks "build once, deploy many," introduces drift
3. **Modifying servers in place** — anti-pattern; immutable deployment is the modern approach
4. **Thinking CI is about deployment** — CI is about *integration and testing*, not shipping to prod
5. **Forgetting the artifact concept** — the artifact is what moves through the pipeline
