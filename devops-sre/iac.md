# Infrastructure as Code (IaC)

Managing infrastructure through code and configuration files rather than manual processes. The foundation of modern, reproducible operations and a key SRE skill (Terraform, Ansible, etc.).

## What Is IaC?

**Defining and provisioning infrastructure through machine-readable files** instead of clicking in a console or SSHing into servers.

```
❌ Manual:  Click through AWS console to create a server
            → Not reproducible, no history, error-prone, "snowflake" servers

✅ IaC:     Write a Terraform file describing the server
            → Version controlled, reviewable, reproducible, automated
```

**Benefits:**
- **Reproducible** — same code produces the same infrastructure every time
- **Version controlled** — infrastructure changes are tracked in git, reviewable in PRs
- **Auditable** — you can see who changed what, when, and why
- **Automatable** — no manual steps, fits into CI/CD
- **Self-documenting** — the code *is* the documentation of what exists

---

## Declarative vs Imperative

The most important conceptual distinction in IaC.

| Approach | You Describe | Tool Figures Out | Examples |
|----------|-------------|------------------|----------|
| **Declarative** | The desired **end state** | *How* to get there | Terraform, CloudFormation, Kubernetes |
| **Imperative** | The **steps** to execute | Nothing (you specify steps) | Bash scripts, Ansible (partly) |

### Declarative

**You describe WHAT you want (the final state), and the tool figures out HOW to achieve it.**

```hcl
# Terraform (declarative) — "I want 3 servers"
resource "aws_instance" "web" {
  count         = 3
  instance_type = "t3.micro"
  ami           = "ami-12345"
}
```

You don't say "create server 1, then create server 2..." You say "I want 3 servers to exist." The tool:
- Checks current state (how many exist now?)
- Calculates the difference (need to create 2 more? delete 1?)
- Makes it match your declaration

**Pros:**
- You don't manage the "how" — less error-prone
- Idempotent by nature (running again just maintains the state)
- The tool handles current-state detection

### Imperative

**You specify the exact STEPS to execute, in order.**

```bash
# Bash (imperative) — "do these steps"
create_server "web-1"
create_server "web-2"
create_server "web-3"
```

You're responsible for the logic: What if web-1 already exists? What if the script ran before? You have to handle all of that yourself.

**Pros:**
- Full control over the exact sequence
- Easier to understand for simple linear tasks

**Cons:**
- You must handle current state yourself
- Not idempotent unless you carefully make it so
- Error-prone for complex infrastructure

### The Mental Model

```
Declarative:  "I want a sandwich"           (describe the goal)
Imperative:   "Get bread, add cheese,       (describe each step)
               add ham, close sandwich"
```

---

## Idempotency

**Running the operation multiple times produces the same result as running it once.** This is *critical* for IaC reliability.

```
Idempotent:      Run once  → 3 servers exist
                 Run again → 3 servers exist (no change, no error)
                 Run again → 3 servers exist (still fine)

NOT idempotent:  Run once  → 3 servers created
                 Run again → 3 MORE servers created (now 6!)
                 Run again → 9 servers! (disaster)
```

**Why it matters:** You need to be able to run your IaC repeatedly — after a failure, during a retry, as part of CI — and trust that it converges to the desired state without duplicating or breaking things.

**Declarative tools are idempotent by design** (they reconcile to the desired state). Imperative scripts must be *carefully written* to be idempotent (check-before-act).

```bash
# NOT idempotent
mkdir /data              # Fails if /data already exists

# Idempotent
mkdir -p /data           # Succeeds whether or not it exists
```

---

## Drift

**When the real infrastructure no longer matches what the code says it should be.** Usually caused by manual changes.

```
Code says:        3 servers, port 443 open
                       ↓
Someone manually:  opens port 8080 via the console (quick fix during incident)
                       ↓
Reality now:       3 servers, ports 443 AND 8080 open
                       ↓
DRIFT: reality ≠ code
```

**Why drift is dangerous:**
- Your code no longer reflects reality
- The next `terraform apply` might *revert* the manual change (removing the needed port 8080)
- Or someone reads the code and makes wrong assumptions
- Reproducing the environment from code produces something different

**How to handle drift:**
- **Detect it:** `terraform plan` shows the difference between code and reality
- **Prevent it:** Restrict manual console access; all changes go through code
- **Fix it:** Either update the code to match reality, or re-apply to force reality back to code

```bash
# Detect drift
terraform plan
# Output: "~ aws_security_group.web will be updated in place
#          - port 8080 will be removed"
# This tells you reality has port 8080 that code doesn't know about
```

---

## State

**A record of what infrastructure currently exists**, maintained by the IaC tool so it knows what it manages.

In Terraform, this is the **state file** (`terraform.tfstate`):
```
State file maps:  your code  ↔  real resources
                  "web" resource  ↔  aws_instance i-0abc123
```

**Why state exists:** When you run `terraform apply`, the tool needs to know: Does this resource already exist? Terraform tracks that in state, so it knows whether to create, update, or do nothing.

```
Without state:  "Should I create this server? I have no idea what exists."
With state:     "State says server i-0abc123 is my 'web' resource. 
                 It exists and matches. No action needed."
```

### State Locking

**Preventing two people from modifying infrastructure at the same time.**

```
Problem:  Alice runs `terraform apply`
          Bob runs `terraform apply` at the same time
              ↓
          Both read state, both make changes
              ↓
          State corruption, conflicting changes, chaos

Solution: State locking
          Alice runs apply → acquires LOCK
          Bob runs apply → "State is locked by Alice, waiting..."
          Alice finishes → releases lock
          Bob proceeds safely
```

**How it works:** State is stored in a shared backend (S3 + DynamoDB, Terraform Cloud, etc.) that supports locking. The first operation locks it; others wait.

**Interview point:** State locking is why you store state remotely (S3, Terraform Cloud) rather than on your laptop — shared, locked, backed up.

---

## IaC Tools (Know the Landscape)

| Tool | Type | Primary Use |
|------|------|-------------|
| **Terraform** | Declarative | Provisioning cloud infrastructure (servers, networks, DBs) |
| **CloudFormation** | Declarative | AWS-specific provisioning |
| **Ansible** | Mostly imperative | Configuration management (installing software, configuring) |
| **Salt** | Config management | Configuration + remote execution |
| **Pulumi** | Declarative | IaC using real programming languages |
| **Helm** | Declarative | Kubernetes package/deployment management |

### Provisioning vs Configuration Management

```
Provisioning:            "Create the servers, network, load balancer"
(Terraform)              → builds the infrastructure

Configuration Management: "Install nginx, set up users, configure files"
(Ansible)                → configures what's inside the servers
```

Often used together: Terraform creates the servers, Ansible configures them.

---

## Interview Scenarios

### Scenario 1: "Explain declarative vs imperative IaC"

**Strong answer:**
```
"Declarative means I describe the END STATE I want, and the tool 
figures out how to get there. In Terraform, I say 'I want 3 servers' 
— I don't write the steps. The tool checks what exists and reconciles.

Imperative means I write the STEPS explicitly, like a bash script: 
'create server 1, create server 2, create server 3.' I'm responsible 
for the logic, including what happens if it ran before.

The big advantage of declarative is idempotency — I can run it 
repeatedly and it just converges to the desired state. With 
imperative, I have to carefully write it to avoid, say, creating 
duplicate servers on a second run.

Terraform is declarative; a raw bash script is imperative. Ansible 
sits in between — it's mostly declarative in intent but task-ordered."
```

### Scenario 2: "What is configuration drift and why is it a problem?"

**Strong answer:**
```
"Drift is when the actual infrastructure no longer matches what the 
code says. It usually happens when someone makes a manual change — 
say, opening a port in the console during an incident.

It's dangerous for a few reasons: First, the code no longer reflects 
reality, so anyone reading it gets the wrong picture. Second, the 
next `terraform apply` might revert that manual change — removing 
something that's actually needed. Third, if I rebuild from code, I 
get a different environment than production.

I'd detect it with `terraform plan`, which shows the diff between 
code and reality. To prevent it, I'd lock down manual access so all 
changes go through code review. The whole point of IaC is that the 
code is the source of truth — drift breaks that guarantee."
```

### Scenario 3: "Why does Terraform need state, and why store it remotely?"

**Strong answer:**
```
"State is how Terraform knows what it already manages. When I run 
apply, it needs to know: does this server exist already? State maps 
my code resources to real-world resources, so it can decide whether 
to create, update, or leave things alone.

I store it remotely — like S3 — for two reasons. First, it's shared: 
my whole team works from the same state, not separate copies on 
laptops. Second, state locking: if two people apply at once without 
locking, they corrupt the state. A remote backend with locking makes 
the second person wait until the first finishes. Plus it's backed up, 
which matters because losing state is painful."
```

---

## Interview Tips

1. **Declarative vs imperative is the key concept** — describe end-state vs steps
2. **Idempotency** — run many times, same result; declarative tools get this for free
3. **Drift = reality diverges from code** — usually from manual changes
4. **State = what the tool thinks exists** — needed to plan create/update/delete
5. **State locking prevents concurrent corruption** — why state lives remotely
6. **Terraform (provision) vs Ansible (configure)** — know the division of labor
7. **The code is the source of truth** — the whole philosophy of IaC

---

## Common Pitfalls

1. **Confusing declarative and imperative** — Terraform describes state, bash describes steps
2. **Writing non-idempotent scripts** — `mkdir` vs `mkdir -p`
3. **Ignoring drift** — manual changes silently break the code-as-truth model
4. **Storing state locally** — breaks team collaboration and locking
5. **Mixing provisioning and configuration** — Terraform builds, Ansible configures
6. **Not version-controlling IaC** — defeats the auditability benefit
