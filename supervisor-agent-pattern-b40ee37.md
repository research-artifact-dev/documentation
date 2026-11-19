[//]: # (loio: b40ee37)

# Chapter 2: The Supervisor Agent Pattern

## 2.1 Motivation: Why a Supervision Tier

A well-built worker agent handles the nominal path reliably. The cases that break
well-built agents are not nominal: unexpected data quality, edge-case inputs, compound
errors that individually pass validation but collectively produce a wrong result, gradual
drift in model behavior. These failure modes share a property — they are hard to detect
from inside the worker agent's own execution loop.

A supervision tier adds an external vantage point. Where the worker agent reasons about
"what is the correct answer to this task", the supervisor reasons about "is this worker
agent behaving correctly". These are different questions, and assigning them to the same
reasoning loop creates a conflict of interest.

The supervisor pattern also provides the mechanism for the escalation path in Chapter 1
section 1.6: Levels 2 and 3 (supervised and blocked) require a component that can
intercept output, enforce thresholds, and route to `sapTaskCenter` or SAP AI Agent Hub
independently of the worker's own self-assessment.

---

## 2.2 Alternatives Considered

Before committing to the supervisor pattern, consider whether a simpler option suffices:

| Option | When it is sufficient | Limitation |
|---|---|---|
| Self-verification only (Ch1 §1.4.3) | Low-risk, read-only, single-step outputs | Same model generates and verifies; no independence; no cross-invocation monitoring |
| Output schema validation (guardrails Ch4) | Numerical bounds, unit consistency, range checks | Deterministic checks only; cannot evaluate semantic correctness or inter-invocation patterns |
| Supervisor pattern (this chapter) | High-risk write-back, compound outputs, production-monitored multi-agent | Additional node overhead; requires shared state store for multi-agent deployments |

The supervisor pattern is appropriate when at least one of the following applies:

1. The agent writes back to a SAP system of record (creates or modifies transactional data)
2. The output is a numerical commitment with regulatory or financial consequences
3. Multiple agents must produce consistent, mutually non-contradictory outputs
4. The deployment requires ongoing quality monitoring by an external party (CIS T&O, customer compliance)
5. The agent's behavior must be auditable independently of the worker's own logs

---

## 2.3 The Three Supervisor Functions

The supervisor is not a single workflow; it is three separate functions implemented
as separate Joule Studio workflows. Each function has a distinct trigger, a distinct
purpose, and distinct data protection requirements. They share a state store (see §2.4).

### 2.3.1 Workflow A — Output Verification and Approval Routing

**What it does**: receives every high-risk worker output before it is written to any
SAP system of record. Applies deterministic checks (Code node), LLM-based semantic
critique (second `sapOrchestration` call with a critic system prompt), and routes
the result to one of four paths: direct commit (both checks confirm), human review
(one or both checks reject or disagree), or autonomous rejection.

**Trigger**: webhook from the worker agent's `sapAgent` callback node — or via
`sapOrchestration` callback pattern when `sapAgent` is not available.

**Four routing outcomes:**

| Outcome | Condition | Action |
|---|---|---|
| BOTH CONFIRM | Deterministic Code node passes AND LLM critic approves | Write output to system of record via `sapMcpClient`; write to audit trail |
| BOTH REJECT | Deterministic Code node fails AND LLM critic rejects | Autonomous rejection; `sapTaskCenter` task with reject reason; write to audit trail |
| DISAGREE | Deterministic Code node and LLM critic produce different verdicts | Human review required; `sapTaskCenter` approval task with both verdicts visible; write to audit trail; suspend output until human decides |
| LOW CONFIDENCE | Worker agent's `outputSchema` confidence field below threshold | Human review required; same routing as DISAGREE |

The DISAGREE state is the most valuable case: it surfaces outputs where the deterministic rule and the
semantic check do not agree, which is exactly the case where human judgment adds the most value.
A supervisor that routes DISAGREE outputs to humans is not failing — it is functioning correctly.

**Why DISAGREE is a signal, not a failure**: the deterministic rule and the LLM critic have different
failure modes. The deterministic rule can pass a semantically wrong output that happens to be
numerically within range; the LLM critic can reject a correct output because the phrasing is unusual.
When they disagree, the overlap of their respective blind spots is exactly the region that benefits from human judgment. Over time, DISAGREE patterns surface systematic misalignments between the
deterministic rules and the actual domain semantics — inputs to both rule refinement and critic
prompt improvement.

### 2.3.2 Workflow B — Telemetry Monitoring and Anomaly Detection

**What it does**: on a scheduled basis (not per-invocation), retrieves operational telemetry
from SAP Cloud ALM and the AI Core usage API, applies a rule set to detect anomalies
in the worker agent's behavior patterns, and routes alerts to the appropriate human.

**Trigger**: Joule Studio `Schedule Trigger` node — configurable interval; typically
hourly for production, daily for EAC.

**Six anomaly rule types (domain-agnostic, deploy-as-is from Starter Kit):**

| Rule type | Signal | Threshold approach |
|---|---|---|
| Error rate spike | Tool call error rate exceeds baseline | Rolling 24h rate vs. p95 baseline; alert if > 2× baseline |
| Tool call count per invocation | Average tool calls per invocation exceeds baseline | Rolling 7-day average vs. EAC-calibrated baseline; alert if > 1.5× |
| Response latency | p95 latency exceeds baseline | Rolling 24h p95 vs. EAC-calibrated p95; alert if > 2× |
| Human override rate | Fraction of Workflow A outputs that humans override | Rolling 7-day rate vs. target; alert if > configured threshold |
| Verification rejection rate | Fraction of Workflow A outputs rejected (both BOTH REJECT and human rejects) | Rolling 7-day rate vs. target; alert if > configured threshold |
| Identical input pattern | Same or structurally identical inputs repeating at abnormal frequency | Duplicate detection with configurable window; alert if same input appears > N times in window |
| Token anomaly (cost extension) | Per-invocation tokens vs. rolling baseline | Runaway detection: single invocation > 5× p99 baseline; daily budget threshold; rising cost per committed output trend |

The token anomaly rule (last row) is the cost monitoring extension from Chapter 8
section 8.5. It is structurally part of Workflow B but serves as the connection
between operational monitoring and the token efficiency framework in Chapter 10.

**Threshold calibration**: all threshold values are Layer 2 parameters — the rule
structure is fixed (Layer 1), the threshold values are set after EAC baseline
observation and documented in the production handover package. The EAC period is
specifically the calibration window: Workflow B runs in monitoring mode (alerts
suppressed or routed to FDE team only) for the first EAC period, establishing the
behavioral baseline before production thresholds are armed.

### 2.3.3 Workflow C — Support Relay and IT Admin Notification

**What it does**: when Workflow A produces a human review task that an end user cannot
resolve, or when Workflow B detects an anomaly that requires IT attention, Workflow C
sanitizes the operational data (strip personal data, retain technical signals), creates
an incident via the SAP Cloud ALM ITSM Adapter API, and notifies the designated IT
admin role.

**Trigger**: webhook from Workflow A (high-severity rejection that bypasses normal
Task Center flow) or from Workflow B (anomaly alert requiring IT action).

**Key properties:**

- **Full deploy-as-is from Starter Kit**: zero per-engagement content. The sanitization
  rules (see §2.5.3) and the Cloud ALM ITSM API call are the same in every engagement.
  Customer tenant ID is injected from the environment configuration; it is not hardcoded.
- **Data protection is built in**: Workflow C is the layer where the sanitization rules
  are applied — it never receives unsanitized personal or business data because Workflows
  A and B only pass operational signals (counts, rates, error codes) to Workflow C.

---

## 2.4 Architecture Design

### 2.4.0 One Agent Per Actor, One Supervisor Per Output Type

The CIS topology design rule is:

- **One worker sub-agent per business Actor** (from the domain primitive vocabulary, Chapter 1 §1.3.4). An Actor maps to a customer persona — procurement officer, plant manager, compliance reviewer. Each Actor's scope is distinct; keeping them in separate agents maintains clear reasoning boundaries and simplifies authorization scoping.
- **One Workflow A instance per output type** (not per worker agent). Two workers that produce structurally similar output — same units, same validation logic, same reference calculation — share one Workflow A. If verification logic diverges significantly between workers, create separate instances.

This topology produces a many-to-one relationship between workers and Workflow A in some deployments (multiple workers, one shared Workflow A differentiated by `agent_identity`), and a one-to-one relationship when output types are distinct. Chapter 7 (Multi-Agent Orchestration) extends this to three formal topologies for multi-agent deployments.

### 2.4.1 Worker-to-Supervisor Communication

Worker agents communicate with the supervisor through one of two patterns. The
choice depends on BAIP platform availability:

**Pattern 1 — sapAgent callback (preferred when available):**
The worker's Joule Studio workflow includes a `sapAgent` callback node. After the
LLM generates an output, the worker pauses execution and posts the output via the
`sapAgent` node to the supervisor's webhook. The supervisor executes Workflow A and
posts an approval or rejection signal back via the `sapAgent` callback. The worker
resumes with the supervisor's decision.

```
[Worker: sapOrchestration: generate output]
        ↓
[Worker: sapAgent: post to Workflow A webhook; wait for callback]
        ↓ (async)
[Supervisor: Workflow A]
  ├── Code node: deterministic checks
  ├── sapOrchestration: LLM critic
  └── [route: BOTH CONFIRM / BOTH REJECT / DISAGREE / LOW CONFIDENCE]
        ↓ (approval signal back to worker)
[Worker: continue if approved; abort if rejected; await human if DISAGREE]
```

**Pattern 2 — sapOrchestration callback (interim pattern):**
When `sapAgent` is not available (EAC-gated; not broadly GA), the worker makes an
explicit HTTP call to Workflow A's webhook (`HTTP Request` node) and polls for the
result or uses a webhook response pattern. This achieves the same logical flow but
requires an additional polling loop in the worker. The worker should implement a
configurable timeout (default: 30 seconds) and fall back to the supervisor fail-safe
rule (§2.7) if no response is received.

The bidirectional A2A protocol resolves this in the longer term: once bidirectional
A2A is GA (H1 2027 target per Chapter 0 constraints), all inter-agent communication
including supervisor callbacks will use standard A2A message passing rather than
webhook/polling patterns. Plan for this migration in the productization package.

### 2.4.2 Supervisor Agent Identity and Separation

The supervisor must have a distinct agent identity from the worker agents it governs:

- **Separate Joule capability bundle**: the supervisor registers as a distinct agent in the
  SAP AI Agent Hub with its own capability name, agent card, and version history.
- **Separate authorization scope**: the supervisor has read access to the BAIP telemetry
  API and write access to `sapTaskCenter`; it does NOT have the worker agent's tool
  access (the worker's SAP system connections). This is the least-privilege principle
  applied to supervisor design — the supervisor cannot accidentally trigger the operations
  it is overseeing.
- **Independent BAIP harness telemetry**: the supervisor's execution traces and the worker's
  execution traces are separate streams in SAP Cloud ALM, with separate agent IDs. This
  makes it possible to audit the supervisor independently of the worker.

### 2.4.3 Shared State Store

Workflows A, B, and C require persistent state to function:

- Workflow A: audit trail of every output decision (committed, rejected, human-overridden)
- Workflow B: rolling metrics baseline (the reference against which anomalies are detected)
- Multi-agent (Chapter 7, Topology 3): the Pending Set table (tracks which workers have
  delivered verified outputs; gates the orchestrator until all workers complete)

This state is stored in a **HANA Cloud database** provisioned in the engagement's BTP
sub-account. Two tables:

| Table | Purpose | Schema |
|---|---|---|
| `SUPERVISOR_AUDIT_TRAIL` | Append-only record of every Workflow A decision | See Chapter 4 §4.7 for full field set + Chapter 8 §8.5 for cost extension fields |
| `SUPERVISOR_BASELINE` | Rolling metrics snapshot for Workflow B anomaly detection | Per-agent-id, per-metric: rolling mean, p95, last-updated timestamp |

For multi-agent deployments (Chapter 7, Topology 3), a third table:

| Table | Purpose |
|---|---|
| `SUPERVISOR_PENDING_SET` | Tracks which worker callbacks have arrived for a given orchestration invocation; gates release to orchestrator until all workers verified |

The shared state store DDL is a Starter Kit artifact (see Chapter 9 §9.4 for status). The audit trail table is the same table required by Chapter 4's precision guardrail design — these are not two separate tables; the audit trail from Chapter 4 and the supervisor's state store audit trail are the same HANA Cloud table, written by Workflow A.

---

## 2.5 Data Protection Design

The supervisor pattern creates a monitoring channel — Workflow B — that must observe
agent behavior without observing personal or business data. This section defines the
three-tier approach.

### 2.5.1 Tier 1: Telemetry Separation by Design

The BAIP managed agent runtime separates execution telemetry from inference content
by design:

- **Business data and LLM reasoning** remain in the customer's BAIP tenant, governed by
  the BAIP tenant isolation model and the customer's data processing agreement with SAP.
- **Operational telemetry** (OpenTelemetry traces, error codes, latency metrics, tool
  call counts) is routed to SAP Cloud ALM — an SAP-operated service, region-bound to
  the customer's contracted data-residency region.

Workflow B queries SAP Cloud ALM for operational telemetry only. It does not access
the LLM inference logs, the content of tool call inputs/outputs, or the user session
content. This is a structural property of the BAIP architecture, not a configuration
choice the FDE team makes.

**Data residency clarification**: operational telemetry stays within the customer's
SAP-operated data-residency boundary (not strictly "within the tenant", but within the
boundary of the customer's contracted SAP Cloud ALM region). Business data and LLM
inference stay local to the BAIP tenant. For sovereign deployments (CN40, NS2, KSA),
confirm that SAP Cloud ALM is available in the sovereign boundary before relying on
Workflow B's telemetry queries — see §3.12 and Chapter 6 §6.8.

### 2.5.2 Tier 2: sapDPI for Personal Data in Tool Call Outputs

For tool calls that may return personal data (customer names, employee IDs, any
personally identifiable information from SAP system of record reads), the BAIP
`sapDPI` (Data Privacy Integration) capability provides automatic detection and
masking before data is processed by the LLM or logged.

FDE teams must configure `sapDPI` explicitly for any tool that reads personal data.
The configuration specifies which fields are personal and what masking rule applies
(hash, redact, generalize). `sapDPI` integration does not happen automatically — it
is a workflow node configuration the FDE Application Engineer must add.

For the supervisor specifically: Workflow A receives output from the worker, which
may already contain personal data passed through from tool call results. The correct
design is to apply `sapDPI` at the worker level (before the output is posted to Workflow A's
webhook), not at the supervisor level. Personal data that reaches Workflow A's input
has already left the worker's controlled environment.

### 2.5.3 Tier 3: Sanitization Rules for Workflow C (Support Relay)

Workflow C relays operational signals to IT admin and potentially to SAP T&O for
support escalations. The sanitization rules define what may be included:

**RETAIN (operational signals — no personal or business data):**
- Timestamp
- Agent ID and version
- Error code and category
- Anomaly type and rule that fired
- Tool call count and latency metrics
- Workflow A routing decision (BOTH CONFIRM / BOTH REJECT / DISAGREE / LOW CONFIDENCE)
- Verification stage verdicts (pass/fail per stage)
- Correlation ID (for trace linking only — not linkable to a customer or user without additional data held in the customer's tenant)

**STRIP (personal and business data — never relay to Workflow C):**
- Input data from the user session
- LLM-generated output content (the agent's recommendation text)
- Tool call inputs and outputs (which contain business data from SAP systems)
- User identifiers (user ID, employee ID, customer number)
- Any field that is a primary or foreign key to a business entity in the customer's
  system of record

The sanitization rules are the Layer 1 component of Workflow C — they are the same
in every engagement and must not be relaxed per-engagement, regardless of customer
preference. Workflow C's value as a support escalation channel depends on its data
protection properties being unconditional.

**Legal basis for Workflow C**: relaying even sanitized operational telemetry to SAP T&O
(CIS, FDE team, support) may require an explicit legal basis under GDPR article 28 (data
processing agreement) depending on the customer's jurisdiction and the nature of the
engagement. The legal team has flagged this — confirm the applicable legal basis during
FDE Discovery for any engagement where Workflow C is in scope, before the engagement
contract is signed. See open question Q7 in `open-questions-3a2265d.md`.

---

## 2.6 Escalation Thresholds

The supervisor escalation thresholds are the per-engagement parameters that govern
when Workflow A routes to human review and when Workflow B triggers an alert.
They must be configured before EAC and documented in the production handover package.

### Workflow A Thresholds

| Threshold | What it governs | Who sets it | When |
|---|---|---|---|
| Approval threshold (absolute) | Outputs above this value require human approval regardless of verification results | Process owner, confirmed in Discovery | Discovery sub-phase 3 (Define) |
| Approval threshold (deviation) | Outputs that deviate more than this percentage from a reference value require human approval | Process owner, confirmed in Discovery | Discovery sub-phase 3 (Define) |
| Confidence threshold | Outputs below this confidence level route to human review | FDE team, calibrated from EAC evaluation | EAC calibration period |

The absolute and deviation thresholds are Layer 3 parameters — engagement-specific,
agreed with the process owner, documented in the Draft Product Design Specs. The
confidence threshold is calibrated from EAC evaluation data: the FDE team observes
the confidence distribution during EAC and sets the threshold at the value that
distinguishes high-confidence correct outputs from low-confidence potentially incorrect ones.

### Workflow B Thresholds

All Workflow B threshold values are calibrated from the EAC baseline:

```
1. Deploy agent to EAC environment.
2. Run Workflow B in monitoring mode (alerts suppressed; FDE team receives reports).
3. Observe agent behavior over the EAC period (target: 2 weeks minimum).
4. Record baseline statistics per metric: mean, p95, distribution shape.
5. Set production thresholds at 2× p95 for rate-based metrics; at 1.5× mean for
   count-based metrics; daily budget threshold agreed with customer IT admin.
6. Document all threshold values in the production handover package.
7. After production go-live, re-calibrate if behavior changes materially within
   the first 30 days.
```

Thresholds set without an EAC baseline — borrowed from a prior engagement or guessed —
generate false positives or miss genuine anomalies. The EAC calibration step is not
optional.

---

## 2.7 Fail-Safe Rule

When the supervisor is unavailable — Workflow A's webhook is unreachable, Workflow B
fails to retrieve telemetry, or the approval callback times out — the fail-safe rule
governs what the worker agent does:

> **If the supervisor is unavailable, the worker agent routes all outputs that would
> normally go through Workflow A directly to `sapTaskCenter` for human review. The
> worker agent does NOT write any output to a system of record until either Workflow A
> approves or a human approves via Task Center.**

The fail-safe is a Joule Studio workflow pattern:

```
[Worker: post to Workflow A webhook]
        ↓
[If: Workflow A response received within timeout?]
    ├── YES → [follow Workflow A routing decision]
    └── NO  → [sapTaskCenter: escalate with reason "supervisor unavailable"]
              [Worker: suspend write-back; await Task Center decision]
```

The fail-safe must be in the worker workflow from the first deployment. Adding it
retrospectively (after a production incident where the supervisor was unavailable and
the worker wrote unsupervised outputs) is a remediation step, not a design choice.

---

## 2.8 LLM Critic Quality: Testing the Supervisor

The LLM critic in Workflow A is the engagement-specific component of the supervisor
(Layer 3). It is also the component most likely to introduce silent quality degradation:
a critic system prompt that approves wrong outputs or rejects correct ones is worse than
no critic at all, because it creates false confidence in the supervision layer.

### What Must Be Tested

| Test type | What to verify |
|---|---|
| Happy path with correct output | Critic returns APPROVE with appropriate reasoning |
| Out-of-range output (numerical) | Critic returns REJECT; reasoning references the specific violation |
| Plausible-but-wrong output (semantically incorrect but numerically in range) | Critic returns REJECT; reasoning explains why the semantic content is wrong |
| Novel edge case (input type not in training) | Critic returns APPROVE (if output is correct) or REJECT (if wrong); reasoning should not hallucinate a constraint that does not exist |
| Adversarial input (prompt injection in worker output) | Critic follows its instructions; does not follow injected instructions; returns APPROVE or REJECT on the actual output |

### Test Cadence

The critic test suite should be run:
- Once, before the supervisor is deployed to EAC (verify the critic prompt works as designed)
- After any change to the critic system prompt
- After any model version change that affects the `sapOrchestration` configuration used in Workflow A
- When Workflow B detects an anomalous DISAGREE rate (unusual disagreement between Code node and critic may indicate critic drift)

### Critic Prompt Design Rules

1. **State the domain explicitly in the opening.** The critic must know what it is evaluating.
   "You are a quality reviewer for [specific domain] outputs in an SAP [specific Industry AI]
   context." Generic system prompts produce generic verdicts.

2. **List the specific constraints the critic enforces.** Do not ask the critic to "evaluate
   quality generally". Enumerate the specific rules: "Output must include [X]. Output must
   not recommend [Y]. If the output references [Z], it must include [reference]."

3. **Define the output format strictly.** The routing logic in Workflow A depends on
   parsing the critic's output. Use a fixed-structure format: `{"verdict": "APPROVE" | "REJECT",
   "reason": "...", "specific_violation": "..." | null}`.

4. **Include few-shot examples for the boundary cases.** The DISAGREE cases — where the
   critic's verdict will differ from the Code node's — are the most important to cover with
   examples. Without examples, the critic may systematically agree with the deterministic rule
   even in cases where it should not.

5. **Test the critic prompt as a unit, separately from the full Workflow A.** Run the critic
   prompt against a fixed set of inputs (correct, wrong, edge case) and verify the verdicts
   before integrating it into the workflow. A critic that cannot be unit-tested is not
   production-ready.

---

## 2.9 Supervisor in Multi-Agent Deployments

Multi-agent orchestration is covered in detail in Chapter 7. The supervisor pattern
extends to multi-agent deployments in two ways.

### Worker-Level Supervision (Topology 2)

Each worker has its own dedicated supervisor (Workflow A). The orchestrator receives
only supervisor-approved outputs. The supervisor is not in the orchestrator's execution
loop — it is in each worker's execution loop.

```
Orchestrator
├── Worker 1 → Supervisor A1 → (approved outputs only) → back to Orchestrator
├── Worker 2 → Supervisor A2 → (approved outputs only) → back to Orchestrator
└── Worker 3 → Supervisor A3 → (approved outputs only) → back to Orchestrator
```

This topology provides maximum isolation: a supervisor failure on one worker does not
affect the others. It is also the most resource-intensive: three separate Workflow A
instances, each calibrated for its worker's specific output type.

### Shared Supervision (Topology 3)

A single supervisor governs all workers. The supervisor tracks which workers have
completed verification using the Pending Set table in the shared state store, and
releases the combined result to the orchestrator only when all workers have been verified.

```
Orchestrator
├── Worker 1 ──┐
├── Worker 2 ──┤─→ Shared Supervisor → (all approved) → back to Orchestrator
└── Worker 3 ──┘
```

The key property: the orchestrator never receives a partial set of approved outputs.
If Worker 2's output is rejected or requires human review, the orchestrator does not
receive Workers 1 and 3's approved outputs independently — it receives nothing until
the full set is resolved. This is the atomic commit principle for multi-agent outputs:
partial approval is not approval.

The shared supervisor also enforces cross-worker consistency checks (Chapter 7 §7.5):
after all workers have submitted verified individual outputs, the supervisor checks
mutual consistency (do the outputs from different workers add up? are units consistent?
are timelines compatible?) before releasing to the orchestrator.

### Supervision at Scale: What Does Not Change

Regardless of topology, the following properties are invariant:

- Workflow A routing logic (BOTH CONFIRM / BOTH REJECT / DISAGREE / LOW CONFIDENCE) applies
  per worker output — the supervisor never aggregates before verifying individually
- Fail-safe rule applies per worker: if the supervisor is unavailable for Worker 2, all
  three workers' outputs route to Task Center
- Audit trail writes are per-output: every worker output decision is independently recorded
- Workflow B monitoring operates at the worker level (agent_id distinguishes workers)

---

## 2.10 Relationship to Joule Studio Implementation

Chapter 5 provides the concrete Joule Studio (n8n) build map for the full supervisor
pattern. The key cross-reference points:

| Design element (this chapter) | Joule Studio implementation (Ch5) |
|---|---|
| Worker-to-supervisor communication | `sapAgent` callback node or `HTTP Request` to webhook |
| Workflow A — Code node verification | `Code` node (JS/Python sandbox); range checks, unit checks, arithmetic cross-checks |
| Workflow A — LLM critic | `sapOrchestration` node with critic system prompt; `outputSchema` with `verdict`, `reason`, `specific_violation` fields |
| Workflow A — human review routing | `sapTaskCenter` node with `customAttributes` mapping |
| Workflow B — scheduled monitoring | `Schedule Trigger` node; `HTTP Request` to SAP Cloud ALM API; `HTTP Request` to AI Core usage API; `Code` node with anomaly rule set |
| Workflow C — support relay | `HTTP Request` to Cloud ALM ITSM Adapter API; `Function` node for sanitization |
| Shared state store | `PostgreSQL` node or `MySQL` node connecting to HANA Cloud; DDL from Starter Kit |
| Supervisor fail-safe | `If` node on timeout condition; `sapTaskCenter` fallback path |

The build sequence (Chapter 5 §5.10) specifies when each workflow element is built
in the FDE sprint: Workflow A skeleton in Week 3–4, Code node verification rules
in Week 5, Workflow B in Week 7–8, Workflow C (if in scope) in Week 9. The sequence
is designed so the supervisor is never deployed before the worker is stable —
calibrating a supervisor against an unstable worker generates noise rather than signal.

---

## 2.11 Data Protection Summary

| Data type | Where it lives | Who can access it | Supervisor access? |
|---|---|---|---|
| User session content (prompts, dialogue) | BAIP tenant | Only within tenant; masked by DPI | No — Workflow B queries operational telemetry only |
| LLM inference outputs (agent reasoning) | BAIP tenant | Only within tenant | No — Workflow A receives worker-processed output only |
| Tool call inputs/outputs (business data) | BAIP tenant | Only within tenant | No — sanitization rules strip these before Workflow C |
| Operational telemetry (error rates, latencies, tool call counts) | SAP Cloud ALM — SAP-operated, within customer's contracted data-residency region | CIS T&O with appropriate access | Yes — Workflow B's monitoring function |
| Audit trail (verification decisions, approval records) | HANA Cloud in customer's BTP sub-account | Customer IT admin; CIS T&O with consent | Yes — Workflow A writes this; it is the supervisor's record |
| Support escalation signals (sanitized operational data) | Cloud ALM ITSM system | Customer IT admin; SAP support with legal basis | Yes — Workflow C creates these records |

**Sovereign deployment note**: in CN40, NS2, and KSA deployments, SAP Cloud ALM
availability within the sovereign boundary must be confirmed before Workflow B can
function. If Cloud ALM is not available in the sovereign region, Workflow B's
telemetry queries must target an alternative endpoint (or Workflow B is deferred).
See Chapter 3 §3.12 and Chapter 6 §6.8 for the sovereign deployment decision guide.

---

## 2.12 Summary

| Design element | Key principle |
|---|---|
| When to use supervisor | High-risk write-back; numerical commitments; multi-agent consistency; auditable governance |
| Three workflows | A (per-output verification) + B (scheduled telemetry monitoring) + C (support relay); each with distinct trigger, purpose, data protection |
| Worker-to-supervisor communication | sapAgent callback (preferred) or sapOrchestration callback pattern; HTTP polling as interim; A2A as long-term standard |
| Supervisor identity | Separate agent identity, separate authorization scope, separate telemetry |
| DISAGREE routing | Most valuable case — surfaces outputs where deterministic rule and semantic critic disagree; route all DISAGREE cases to human review |
| Fail-safe | Supervisor unavailable → ALL outputs to Task Center; never to system of record |
| Data protection tiers | 1: telemetry separation by BAIP architecture; 2: sapDPI at worker level; 3: sanitization rules in Workflow C |
| Threshold calibration | EAC baseline observation → production thresholds; not borrowed from prior engagements |
| Critic prompt quality | Unit-test the critic independently; enumerate specific constraints; use fixed output format; cover boundary cases with examples |
| Multi-agent | Worker-level supervision (Topology 2) or shared supervision (Topology 3); atomic commit principle: partial approval is not approval |
