<!-- loio67417bb9efc147b1b8e722328394ebb0 -->

# Supervisor Agent Pattern

The supervisor agent pattern implements SAP's Supervision tier for enterprise-grade agentic systems, providing business-logic-level quality enforcement and controlled write-back for Industry AI FDE deployments. Use this pattern to prevent wrong outputs from being committed to regulated systems of record, detect cross-invocation drift, and establish a data-minimizing escalation path to SAP support.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_supervision_tier"/>

## The Supervision Tier and Why It Exists

SAP's AI-Native North Star Architecture defines a three-tier AI defense model for enterprise-grade agentic systems:

-   **Foundation tier** — zero-trust authentication, traffic inspection, prompt-injection detection, behavioral analytics, cryptographic agent credentials. Provided by the BAIP platform; always on.
-   **Supervision tier** — deterministic control agents that monitor, sanitize, and authorize other AI agents. Input validation, output filtering, anomaly detection, human oversight for high-risk decisions. Must be explicitly designed and deployed.
-   **Automation tier** — AI agents scaling security operations; continuous policy audits, risk assessment, identity management.

The Foundation tier handles platform-level threats. The Automation tier handles organizational-scale security operations. The Supervision tier handles the gap between the two: **business-logic-level quality and safety for individual agent workflows**, within the customer's own environment, without requiring external data movement.

The supervisor agent is the CIS implementation of the Supervision tier for Industry AI FDE engagements. It is not a replacement for the Foundation tier (which the BAIP platform harness provides automatically) or for the [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: governance capabilities (which operate at IT-admin level). It is the layer that has visibility into business-logic correctness in a way that neither the platform nor the IT admin can provide.

**The supervisor also serves as an autonomous CIS proxy on the customer tenant — in deployments where CIS has no default access.** This applies specifically to **BYOA runtimes on the customer's own BTP subaccount and to sovereign-cloud deployments**: there, the agent runs in the customer's own environment and CIS has no default access for monitoring. The supervisor bridges this gap — it performs the monitoring and intervention that CIS would otherwise need direct access to do — and only escalates to CIS when it has exhausted what it can handle autonomously. This design reduces the need for permanent CIS access to customer tenants to genuinely exceptional cases, without sacrificing operational visibility.

Under the **SMJ default** for new engagements (the SAP-managed orchestrator, typically paired with a low-code Joule Studio 2.0 runtime), the agent runs on SAP-managed infrastructure, so telemetry and governance are already reachable through the SAP AI Agent Hub and the managed harness. The supervisor's proxy function is therefore most load-bearing for BYOA and sovereign runtimes; on the SMJ-managed path it is primarily a business-logic quality and write-gate layer rather than an access bridge. See the [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md) for the full CIS CoE operations model that builds on this pattern.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_alternatives"/>

## Alternative Approaches and When Each Is Sufficient

Before committing to a supervisor agent, consider the full range of design alternatives — what each covers, where it falls short, and under what conditions it is genuinely sufficient.

The organizing question for each alternative: *what failure mode does it prevent, and how late is too late to catch it?* For irreversible writes — batch release quantities committed to a regulated system of record, financial postings, physical dispatches — post-hoc detection is not acceptable. The design must prevent the wrong output from being committed in the first place.

### Alternative A: Inline Self-Verification Only

A second validation step within the worker agent's own workflow, before write-back. The worker runs a Code node check and a second `sapOrchestration` critic call, then routes itself to write-back or Task Center based on the result.

**What it covers**: output validation for a single invocation at the write-back moment.

**What it does NOT cover**:

-   Cross-invocation patterns, drift over time, or repeated failure modes
-   Platform-level anomalies (error rate spikes, latency outliers)
-   Any issue that occurs after write-back
-   The CIS proxy function — the worker still controls the write-back decision

**Critical limitation**: the worker can be modified to bypass the verification step — it is code inside the same workflow. The quality gate is a design convention, not an architectural enforcement. For regulated outputs or high-value transactions, this distinction matters.

**Sufficient when**: the risk is concentrated at the write-back moment, the output can be verified inline, cross-invocation monitoring is not required, and the consequences of a bypassed gate are acceptable.

### Alternative B: SAP AI Agent Hub Governance Only

The [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: Observe & Analyze and Secure & Govern capabilities provide IT-admin-level visibility: agent registry, access control, lifecycle management, usage telemetry, policy enforcement.

**What it covers**: who is running which agents, with what permissions, at what frequency; policy enforcement and lifecycle events (deactivation, version rollback).

**What it does NOT cover**:

-   Whether individual agent outputs are correct for the business process
-   Business-logic quality of any kind — a batch release agent systematically recommending quantities 15% below correct values will not trigger any Agent Hub alert

**Critical limitation**: the Agent Hub operates at the IT-governance layer. It is authoritative for "is this agent authorized to run and does it comply with policy?" It has no visibility into "is this agent producing the right answer for this business process?"

**Sufficient when**: the primary concern is IT governance and access control, not business-logic quality. Can be used in combination with Alternative A for a lightweight two-layer approach.

### Alternative C: Human-in-the-Loop on Every Output

Every agent output goes through a `sapTaskCenter` approval step before write-back. Humans review and approve or reject each output individually.

**What it covers**: all errors — humans catch everything, by definition.

**Critical limitation**: this is not a quality pattern for a mature deployment; it is a safe mode for situations where confidence in the agent has not yet been established.

**Sufficient when**: during the first weeks of a deployment before the behavioral baseline is established, or for specific output categories where regulatory requirements mandate human sign-off on every decision regardless of agent confidence.

### Alternative D: Asynchronous Batch Evaluation

Rather than intercepting outputs in real-time, run a periodic evaluation job (nightly or weekly) that scores a sample of already committed outputs against defined quality criteria. The [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: Agent Evals & Assessment capability supports this pattern.

**What it covers**: quality trend analysis, ground-truth dataset construction, model drift detection, low operational overhead.

**Critical limitation**: too late for irreversible writes. By definition, outputs have already been committed when the eval runs.

**Where it genuinely adds value**: as a complement to the supervisor pattern, not a replacement. Running async batch eval alongside the supervisor builds the ground-truth dataset that the continuous improvement loop in the [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md) consumes.

### Alternative E: Rule-Based Policy Engine Only

Implement only the deterministic verification layer — a code-based gate — without the LLM critic and without telemetry monitoring.

**What it covers**: detectable, enumerable failure modes; fully auditable and explainable verdicts.

**What it does NOT cover**: contextual plausibility that rules cannot express; drift detection; CIS proxy function.

**Interesting regulatory angle**: in some regulated contexts (GxP, financial controls), a pure rule-based gate may be *preferred* over an LLM critic precisely because it is fully auditable. When regulatory sign-off on the verification logic itself is required, a rule-based engine is more defensible.

**Sufficient when**: all meaningful failure modes can be enumerated and expressed as rules; regulatory auditability of the verification logic is a hard requirement; or as a starting point before the LLM critic is added during the EAC calibration phase.

### Alternative F: Confidence Gating Only

Rely entirely on the agent's own confidence score — above threshold → write-back, below threshold → escalate. No independent verification stage.

**Critical limitation**: LLM confidence scores are unreliable as a sole quality gate. Large language models are well-documented to produce high-confidence incorrect outputs — the phenomenon of "hallucination with certainty" is precisely the failure mode that confidence gating alone cannot catch.

Confidence is a useful supplementary routing signal within a multi-stage verification stack (see [Precision Guardrails](precision-guardrails-ae75c89.md)) — a low confidence score adds a reason to escalate even when deterministic checks pass. It is not a standalone quality gate for write-back agents.

### Alternative G: Downstream Reconciliation (Optimistic Write + Post-Hoc Correction)

The agent writes optimistically — without a quality gate — and a reconciliation job runs periodically to detect anomalies in committed data.

**Critical limitation**: this pattern assumes errors are reversible and cheap to correct. For Industry AI domains — Regulated Manufacturing, Commodity Management, Revenue Growth Management — that assumption does not hold.

### Alternative H: Multi-Model Consensus (Ensemble Approach)

Run the same task through N independent LLMs or agent configurations simultaneously. Commit only when their outputs agree within a defined tolerance; escalate on disagreement.

**What it covers**: reduces the probability of confident-but-wrong outputs; disagreement between models is itself an informative signal.

**Cost constraint**: the AI Units billing model makes the cost implication concrete. Three Advanced-tier LLM calls per invocation instead of one changes the cost-per-committed-output metric significantly. For high-frequency agents, ensemble approaches may consume the efficiency savings that justify the agent in the first place.

**Where it genuinely adds value**: for extremely high-stakes, low-frequency decisions where the cost of N× inference is small relative to the cost of an error.

### Alternative I: Asynchronous Human Review (Non-Blocking)

The agent writes back immediately; a queue of recent decisions is routed to human reviewers asynchronously.

**What it covers**: human oversight without adding latency to the agent's critical path.

**Critical limitation**: the output is committed before human review. This shares the fundamental limitation of Alternatives G and D: if the write is irreversible, post-hoc review cannot undo the consequences.

### Summary: Comparison Across All Alternatives

**Table: Approach Comparison**

<table>
<tr>
<th valign="top">

Alternative

</th>
<th valign="top">

Prevents Wrong Commits?

</th>
<th valign="top">

Detects Drift?

</th>
<th valign="top">

CIS Proxy?

</th>
<th valign="top">

Key Constraint

</th>
</tr>
<tr>
<td valign="top">

A: Inline self-verification

</td>
<td valign="top">

Partially (worker controlled)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Worker can bypass gate

</td>
</tr>
<tr>
<td valign="top">

B: Agent Hub governance only

</td>
<td valign="top">

No

</td>
<td valign="top">

No (IT-level only)

</td>
<td valign="top">

No

</td>
<td valign="top">

No business-logic quality visibility

</td>
</tr>
<tr>
<td valign="top">

C: Human on every output

</td>
<td valign="top">

Yes

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Eliminates automation efficiency

</td>
</tr>
<tr>
<td valign="top">

D: Async batch eval

</td>
<td valign="top">

No (post-hoc only)

</td>
<td valign="top">

Yes (trend)

</td>
<td valign="top">

No

</td>
<td valign="top">

Too late for irreversible writes

</td>
</tr>
<tr>
<td valign="top">

E: Rule-based engine only

</td>
<td valign="top">

Yes (known failure modes)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Cannot catch contextual errors

</td>
</tr>
<tr>
<td valign="top">

F: Confidence gating only

</td>
<td valign="top">

No (confident errors pass)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

LLM confidence scores are unreliable

</td>
</tr>
<tr>
<td valign="top">

G: Downstream reconciliation

</td>
<td valign="top">

No (optimistic write)

</td>
<td valign="top">

Partially

</td>
<td valign="top">

No

</td>
<td valign="top">

Irreversible writes cannot be corrected

</td>
</tr>
<tr>
<td valign="top">

H: Multi-model consensus

</td>
<td valign="top">

Yes (for enumerable disagreements)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

N× token cost; shared systematic failures

</td>
</tr>
<tr>
<td valign="top">

I: Async human review

</td>
<td valign="top">

No (post-hoc)

</td>
<td valign="top">

No

</td>
<td valign="top">

No

</td>
<td valign="top">

Does not prevent committed errors

</td>
</tr>
<tr>
<td valign="top">

**Supervisor pattern**

</td>
<td valign="top">

**Yes (architecturally enforced)**

</td>
<td valign="top">

**Yes**

</td>
<td valign="top">

**Yes**

</td>
<td valign="top">

**Build complexity; Workflow A per output type**

</td>
</tr>
</table>

### When the Supervisor Agent Is Needed

The supervisor pattern is the only approach that simultaneously prevents wrong outputs from being committed through architectural enforcement — the **committer pattern**, where the worker holds no system-of-record write scope and a separate committer identity performs the write — detects cross-invocation patterns and drift without requiring CIS access to the customer tenant, and provides a data-minimizing escalation path to SAP support.

The supervisor agent is the right choice when:

-   The agent is deployed at production scale with significant autonomy
-   Business-logic quality cannot be fully captured by inline self-verification alone
-   Cross-invocation patterns matter (drift, anomalies, repeated failure modes)
-   Data protection requires that telemetry monitoring stays within the customer's tenant
-   Write-back actions are irreversible or regulated

**Mixed-stakes engagements**: a realistic FDE engagement typically contains agents with different risk profiles — one or two high-stakes write-back agents alongside several advisory or reporting agents. Scope the supervisor to the write-back risk surface within the engagement, not to every agent uniformly.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_three_functions"/>

## The Supervisor Agent: Three Functions

The supervisor is not a single monolithic agent. It is a set of three distinct functions, each with a different trigger, execution frequency, and escalation path. Decomposing them into separate [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio — low-code build environment for SAP AI agents, workflows, and tools.") :arrow_upper_right: workflows makes each function independently deployable, testable, and governed.

### Function 1: Inline Quality Relay

**Purpose**: receive the worker agent's output before it is committed to the system of record, run multi-stage verification, and route to write-back or escalation.

**Trigger**: each worker agent invocation (via callback from async worker call).

**Execution pattern**:

```
Worker agent invoked (sapAgent, blocking=false, callbackUrl=supervisor/callback)
         ↓
Supervisor Workflow A receives callback
         ↓
[Stage 1 — Deterministic check: Code node or sapMcpClient calculation]
[Stage 2 — LLM critic: sapOrchestration with critic system prompt]
         ↓
Decision routing (If/Switch node):
    ├── BOTH CONFIRM  → commit: supervisor/committer identity writes (sapMcpClient / OData)
    ├── BOTH REJECT   → escalate: sapTaskCenter with full context
    ├── DISAGREE      → escalate: sapTaskCenter (uncertainty requires human judgment)
    └── DETERMINISTIC UNAVAILABLE → LLM verdict only: route on the LLM-critic confidence threshold
```

The worker agent cannot write to the system of record directly — it holds no write scope at all. It is scoped **read/recommend-only** and returns a **structured proposal** (JSON: the proposed values, the supporting context, and the worker's own confidence) to the supervisor callback. On BOTH CONFIRM, the **supervisor/committer identity** — which holds the write scope the worker lacks — performs the write itself; the worker is never re-invoked to write.

This is the **committer pattern**, and it is what makes the gate *architecturally enforced* rather than a design convention:

-   The worker's effective permission is `min(user, agent)`; with no write scope on the agent identity, the **SAP Agent Gateway denies a direct worker write at the API-call level** — the worker cannot bypass the supervisor even if its own workflow were modified.
-   The write is attributed in the audit trail to the **committer identity**, not the worker.
-   An optional **S/4HANA RAP instance-authorization backstop** rejects the write unless the target record is in an approved state — a third independent gate.

### Function 2: Telemetry Monitor

**Purpose**: continuously monitor the BAIP managed runtime harness telemetry within the customer's tenant, detect anomalies and drift across invocations, and trigger escalation when patterns exceed defined thresholds.

**Trigger**: Schedule Trigger (every 5–15 minutes depending on agent call volume).

**Execution pattern**:

```
Supervisor Workflow B fires on schedule
         ↓
HTTP Request → SAP Cloud ALM pull APIs (hm provider; SAP-operated, region-bound tenant)
         ↓
Code node: rule-based anomaly classification
  - Error rate vs. rolling baseline
  - Tool call count deviation (agent taking unusual paths)
  - Response time percentile drift
  - Repeated identical inputs (possible loop condition)
  - Unusual output distribution (e.g., all outputs above threshold)
         ↓
Decision routing:
    ├── No anomaly → log baseline update, no action
    ├── Anomaly — internal  → sapTaskCenter: alert customer IT admin
    └── Anomaly — SAP support needed → trigger Workflow C (Support Relay)
```

> ### Note:  
> The anomaly classifier in Workflow B must be **rule-based, not LLM-based**. Sending raw BAIP traces to an LLM for classification would route business data through a model, re-introducing the exact privacy concern the intra-tenant design was meant to prevent. Rules, thresholds, and code-based pattern matching are the correct tools here.

### Function 3: Support Relay

**Purpose**: when Workflow B identifies an anomaly that requires SAP support engagement, sanitize the relevant trace extract, create a support ticket, and notify the customer's IT team — without exporting raw operational data.

**Trigger**: event published by Workflow B when anomaly severity exceeds the external engagement threshold.

**Execution pattern**:

```
Supervisor Workflow C receives anomaly event from Workflow B
         ↓
Retrieve specific trace from SAP Cloud ALM (the anomalous invocation only)
         ↓
Sanitization pass (Code node or sapOrchestration with enableMasking):
  - Remove PII via DPI entity masking
  - Remove operational business data (order IDs, material numbers, quantities)
  - Retain: agent ID, tool call sequence, error codes, timing, exception messages
         ↓
SAP Cloud ALM ITSM Adapter API → create support ticket
  (attachment: sanitized trace extract, severity, agent identity, customer tenant ID)
         ↓
HTTP notification → customer IT admin
```

The customer controls what gets exported by defining the sanitization rules in Workflow C. The raw trace is never attached to the outbound support ticket.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_architecture"/>

## Supervisor Architecture in Joule Studio

### Topology Rule: One Actor, One Agent

When deriving agent topology from customer discovery, a single deterministic rule governs the initial topology: **each customer persona (Actor) gets its own dedicated specialist agent. No merging, no sharing.**

A single MasterSupervisorAgent (or equivalent orchestration supervisor) routes between specialist agents based on user intent and routing descriptions. This rule enforces separation of concerns: each specialist agent handles only the domain its Actor works in, making routing deterministic and avoiding mixed-context responses.

**Important distinction**: the orchestration supervisor (routes intent to the correct specialist) is distinct from the governance supervisor described here (monitors, guards, validates). A production deployment typically has both:

**Table: Supervisor Types**

<table>
<tr>
<th valign="top">

Supervisor Type

</th>
<th valign="top">

Function

</th>
<th valign="top">

Position in Architecture

</th>
</tr>
<tr>
<td valign="top">

Orchestration supervisor (MasterSupervisorAgent)

</td>
<td valign="top">

Routes user intent to the correct specialist agent

</td>
<td valign="top">

Entry point; upstream of all specialist agents

</td>
</tr>
<tr>
<td valign="top">

Governance supervisor (this topic)

</td>
<td valign="top">

Monitors output quality, guards write-backs, validates results

</td>
<td valign="top">

Workflow A: per write-back agent (1:1); Workflow B: system-wide across all agents (1:N)

</td>
</tr>
</table>

The governance supervisor's three workflows have different scoping rules:

**Table: Workflow Scope**

<table>
<tr>
<th valign="top">

Workflow

</th>
<th valign="top">

Natural Scope

</th>
<th valign="top">

Relationship

</th>
</tr>
<tr>
<td valign="top">

Workflow A — Inline quality relay

</td>
<td valign="top">

Per write-back agent (or output type)

</td>
<td valign="top">

1:1 — each agent doing irreversible write-backs has its own Workflow A

</td>
</tr>
<tr>
<td valign="top">

Workflow B — Telemetry monitor

</td>
<td valign="top">

Entire topology

</td>
<td valign="top">

1:N — one monitoring loop watches all agents' telemetry

</td>
</tr>
<tr>
<td valign="top">

Workflow C — Support relay

</td>
<td valign="top">

Triggered by Workflow B

</td>
<td valign="top">

On-demand; inherits Workflow B's system-wide scope

</td>
</tr>
</table>

### Worker-Supervisor Communication

The `CUSTOM.sapAgent` node provides the mechanism for inter-agent communication. For the supervisor pattern, `blocking: false` is the correct choice. It decouples the worker's execution time from the supervisor's evaluation time, and allows multiple workers to run concurrently with a single supervisor receiving their callbacks.

**Table: sapAgent Node Parameters (v1.4.9)**

<table>
<tr>
<th valign="top">

Parameter

</th>
<th valign="top">

Type

</th>
<th valign="top">

Default

</th>
<th valign="top">

Purpose

</th>
</tr>
<tr>
<td valign="top">

`agentName`

</td>
<td valign="top">

options

</td>
<td valign="top">

—

</td>
<td valign="top">

Select the SAP Agent to invoke

</td>
</tr>
<tr>
<td valign="top">

`blocking`

</td>
<td valign="top">

boolean

</td>
<td valign="top">

`true`

</td>
<td valign="top">

`false` = fire-and-release; requires `callbackUrl`

</td>
</tr>
<tr>
<td valign="top">

`callbackUrl`

</td>
<td valign="top">

string

</td>
<td valign="top">

—

</td>
<td valign="top">

Webhook URL for async result delivery (non-blocking only)

</td>
</tr>
<tr>
<td valign="top">

`sessionId`

</td>
<td valign="top">

string

</td>
<td valign="top">

—

</td>
<td valign="top">

Correlates callbacks when multiple workers are in flight

</td>
</tr>
<tr>
<td valign="top">

`customContext`

</td>
<td valign="top">

JSON

</td>
<td valign="top">

`{}`

</td>
<td valign="top">

Additional context passed to the agent

</td>
</tr>
<tr>
<td valign="top">

`timeout`

</td>
<td valign="top">

number

</td>
<td valign="top">

180,000ms

</td>
<td valign="top">

Max 300,000ms (5 min); applies in blocking mode only

</td>
</tr>
</table>

### Decomposition into Three Workflows

**Table: Workflow Decomposition**

<table>
<tr>
<th valign="top">

Workflow

</th>
<th valign="top">

Trigger

</th>
<th valign="top">

Primary Nodes

</th>
<th valign="top">

Execution Frequency

</th>
</tr>
<tr>
<td valign="top">

A — Inline Quality Relay

</td>
<td valign="top">

Webhook (worker callback)

</td>
<td valign="top">

Code / sapMcpClient (verify), sapOrchestration (LLM critic), If, sapTaskCenter, sapMcpClient / OData (committer write)

</td>
<td valign="top">

Per worker invocation

</td>
</tr>
<tr>
<td valign="top">

B — Telemetry Monitor

</td>
<td valign="top">

Schedule Trigger

</td>
<td valign="top">

HTTP Request (SAP Cloud ALM pull APIs), Code (rule-based classifier), If, sapTaskCenter, n8n workflow call (→ Workflow C)

</td>
<td valign="top">

Periodic (5–15 min)

</td>
</tr>
<tr>
<td valign="top">

C — Support Relay

</td>
<td valign="top">

Called by Workflow B

</td>
<td valign="top">

Code (sanitize), HTTP Request (SAP Cloud ALM ITSM Adapter API), HTTP Request (notification)

</td>
<td valign="top">

On demand (anomaly events only)

</td>
</tr>
</table>

### Shared State

The three workflows need a common data store for the rolling baseline used by Workflow B's anomaly classifier, pending escalations from Workflow A, and correlation between worker `sessionId` and verification status.

In BAIP, this shared state can be held in:

-   A SAP HANA Cloud table (persistent, queryable, supports concurrent writes)
-   A SAP Build Process Automation work item store
-   A BAIP Key-Value store (for lightweight, ephemeral state)

For FDE engagements, SAP HANA Cloud is the most consistently available option across Industry AI target customers.

### LoopCraft — KG-Enriched Execution Cycle

For agents with access to a Knowledge Graph (HANA KG, EKX, or the CIS Context Layer KG), the recommended execution pattern is a three-step enriched cycle:

**Query KG → Execute Action → Verify Result**

-   **Query KG**: before executing, the agent retrieves relevant context from the knowledge graph — entity attributes, business constraints, prior decisions about related entities, and related process state.
-   **Execute Action**: the agent executes the tool call or decision with full KG context available.
-   **Verify Result**: the agent checks the result against KG constraints or expected values before surfacing it to the user or writing it back to the system of record.

This cycle operates as an **internal quality gate within the agent's own execution** — distinct from the governance supervisor pattern (which monitors from outside). The supervisor remains the architectural enforcement layer; LoopCraft is the agent's own discipline during execution.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_data_protection"/>

## Data Protection Design

### What Stays in the Tenant

The following data never leaves the customer's BAIP tenant under normal operation:

-   Raw BAIP harness traces (agent execution traces, tool call inputs/outputs)
-   Worker agent output before verification approval
-   LLM call history (prompts, responses) from both worker and supervisor
-   Business data used as grounding context (order data, material data, etc.)
-   Intermediate verification results (stage 1 and stage 2 verdicts)
-   Human decisions from SAP Task Center

The one exception is the telemetry pull path to **SAP Cloud ALM**: it moves data out of the customer's BTP subaccount into the SAP-operated Cloud ALM tenant. This telemetry is **stored in the SAP Cloud ALM tenant provisioned in your contracted data-residency region (SAP-operated, region-bound)**.

### What Can Leave the Tenant

Data leaves the tenant in exactly two cases:

1.  **Sanitized trace extract** — only when Workflow C fires, only for invocations where an anomaly was confirmed, only after the sanitization pass.
2.  **Escalation metadata** — `sapTaskCenter` tasks should not contain business-specific data; they should contain agent identity, severity, and the verification verdict only.

### Sanitization Rules

**Table: Trace Field Sanitization Rules**

<table>
<tr>
<th valign="top">

Trace Field

</th>
<th valign="top">

Rule

</th>
</tr>
<tr>
<td valign="top">

Error message / exception text

</td>
<td valign="top">

Retain as-is (structural, not business data)

</td>
</tr>
<tr>
<td valign="top">

Tool call name (e.g., `get_purchase_order`)

</td>
<td valign="top">

Retain as-is (reveals capability, not data)

</td>
</tr>
<tr>
<td valign="top">

Tool call parameters (e.g., PO number, vendor ID)

</td>
<td valign="top">

**Strip** — replace with `[REDACTED]`

</td>
</tr>
<tr>
<td valign="top">

LLM response content

</td>
<td valign="top">

**Strip** — may contain business data, reasoning

</td>
</tr>
<tr>
<td valign="top">

LLM system prompt

</td>
<td valign="top">

**Strip** — may reveal internal business logic

</td>
</tr>
<tr>
<td valign="top">

Agent session ID / correlation ID

</td>
<td valign="top">

Retain (needed for support investigation)

</td>
</tr>
<tr>
<td valign="top">

Timing / latency data

</td>
<td valign="top">

Retain (needed for performance diagnosis)

</td>
</tr>
<tr>
<td valign="top">

Agent identity (agent name, version)

</td>
<td valign="top">

Retain

</td>
</tr>
<tr>
<td valign="top">

Tenant ID

</td>
<td valign="top">

Retain (SAP support needs this)

</td>
</tr>
</table>

These rules should be reviewed and approved by the customer's data protection officer before Workflow C is deployed.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_thresholds"/>

## Escalation Thresholds and Decision Logic

The supervisor must have clearly defined thresholds to avoid two failure modes: **over-escalation** (humans overwhelmed by false alarms) and **under-escalation** (real issues pass undetected). Thresholds should be calibrated per agent, not globally.

### Inline Quality Relay (Workflow A)

**Table: Workflow A Decision Logic**

<table>
<tr>
<th valign="top">

Condition

</th>
<th valign="top">

Action

</th>
</tr>
<tr>
<td valign="top">

Both stages confirm

</td>
<td valign="top">

Approve write-back; log to baseline

</td>
</tr>
<tr>
<td valign="top">

Both stages reject

</td>
<td valign="top">

Block write-back; escalate to named process owner via sapTaskCenter

</td>
</tr>
<tr>
<td valign="top">

Stages disagree

</td>
<td valign="top">

Block write-back; escalate with both verdicts visible to reviewer

</td>
</tr>
<tr>
<td valign="top">

LLM critic confidence below threshold (default 0.7)

</td>
<td valign="top">

Treat as DISAGREE; escalate. Calibrate the threshold during the EAC phase

</td>
</tr>
<tr>
<td valign="top">

Deterministic check unavailable

</td>
<td valign="top">

Proceed on LLM verdict alone; log the gap

</td>
</tr>
<tr>
<td valign="top">

Worker timed out / error

</td>
<td valign="top">

Block write-back; escalate with error context

</td>
</tr>
</table>

### Telemetry Monitor (Workflow B)

**Table: Workflow B Threshold Defaults**

<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
<th valign="top">

Action

</th>
</tr>
<tr>
<td valign="top">

Error rate

</td>
<td valign="top">

> 3× rolling 7-day average for 3 consecutive periods

</td>
<td valign="top">

IT admin alert

</td>
</tr>
<tr>
<td valign="top">

Tool call count per invocation

</td>
<td valign="top">

> 2× p95 baseline for 5 consecutive invocations

</td>
<td valign="top">

IT admin alert + log investigation flag

</td>
</tr>
<tr>
<td valign="top">

Response latency

</td>
<td valign="top">

> 5× p95 baseline for 2 consecutive invocations

</td>
<td valign="top">

IT admin alert

</td>
</tr>
<tr>
<td valign="top">

Human override rate (from Task Center)

</td>
<td valign="top">

> 40% over 7-day window

</td>
<td valign="top">

Process owner review request

</td>
</tr>
<tr>
<td valign="top">

Verification rejection rate (from Workflow A)

</td>
<td valign="top">

> 20% over 24-hour window

</td>
<td valign="top">

Process owner review + Workflow C candidate

</td>
</tr>
<tr>
<td valign="top">

Repeated identical input pattern

</td>
<td valign="top">

Same input parameters 3× within 5 minutes

</td>
<td valign="top">

Possible loop — immediate IT admin alert

</td>
</tr>
</table>

These thresholds are starting points. Calibrate them during the EAC phase using observed baseline behavior before GA deployment.

### Day-1 Cold-Start Guidance

Every threshold above references rolling baselines and p95 values that require historical data to be meaningful. A new deployment has none.

**For the initial observation period — Day 1 through the first 50+ invocations**:

-   **Workflow A (Inline Quality Relay)**: active from day one. It makes per-invocation decisions without requiring a baseline.
-   **Workflow B (Telemetry Monitor)**: use static absolute thresholds rather than baseline-relative ones. Example starting values: error rate > 15% absolute, human override rate > 60% absolute. Switch to baseline-relative thresholds once the observation period is complete.
-   **Anomaly detection alerts**: disabled until 50+ invocations have been recorded, or 5+ business days of operation — whichever is longer for low-frequency agents.

For agents with fewer than ~200 weekly invocations, extend the initial observation period to 2–3 weeks before enabling baseline-relative thresholds.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_governance"/>

## The Supervisor's Own Governance

The supervisor agent is itself a BAIP-managed agent with a first-class identity, scoped authorizations, and its own telemetry.

The supervisor's governance requirements:

-   **Agent identity**: separate application registration in BAIP for each supervisor workflow (A, B, C). Each has its own service credentials and authorization policies.
-   **Least-privilege**: the supervisor/committer identity holds read access to the SAP Cloud ALM telemetry pull path, write access to SAP Task Center, **and** the system-of-record write scope needed to commit approved outputs. The **worker** holds no system-of-record write scope at all.
-   **Supervisor observability**: the supervisor's own telemetry is visible in the SAP AI Agent Hub, just like the worker's.
-   **Supervisor deactivation**: if the supervisor becomes unavailable, the default behavior of the worker is to route all outputs to human review (`sapTaskCenter`), not to write-back autonomously. The fail-safe is always toward more human oversight, not less.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_limitations"/>

## Limitations and Honest Constraints

**Deterministic verification requires that the algorithm exists**: for genuinely open-ended or judgment-based tasks (e.g., summarizing a maintenance report), there is no algorithmic ground truth. In these cases, the LLM critic remains the only automated check, and its independence limitation applies.

**The supervisor cannot verify what it cannot observe**: the supervisor sees the agent's inputs, outputs, and tool call traces. It cannot observe what the agent "thinks" — implicit reasoning failures may pass both verification stages if neither check is designed to catch that specific failure mode.

**Callback architecture adds latency**: the non-blocking worker-supervisor pattern introduces a round-trip. For high-frequency, low-stakes operations this latency may be unacceptable. In those cases, inline self-verification (see [Agent Quality Framework](agent-quality-framework-5f15077.md)) is the right choice, and the supervisor monitors telemetry in the background without being on the critical path.

**LLM critic quality depends on prompt engineering**: the LLM critic in Workflow A (Stage 2 verification) is only as good as its system prompt and the examples provided. The critic prompt is itself a quality artifact that requires its own test suite.

**LLM critic non-determinism**: LLM outputs are probabilistic. Given identical inputs, the critic can return different verdicts across invocations — particularly for outputs near the plausibility boundary. Mitigations: set temperature to 0 in the critic's `sapOrchestration` call; design the critic prompt to require explicit reasoning before the verdict field.

**The support relay requires contractual alignment**: Workflow C creates a data flow from the customer's tenant to SAP support systems. Even with sanitization, this may trigger obligations under the Collaboration Agreement or require explicit consent. The legal review for Workflow C must be completed during the FDE Discovery phase.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_fde_implications"/>

## FDE Delivery Implications

**Discovery**:

-   The TO-BE process flow should explicitly mark where the supervisor intercepts the worker's output.
-   The supervisor's three functions should be scoped explicitly: does this engagement require all three, or only Workflow A? Workflows B and C add complexity and governance overhead; they are not always warranted for the MVP.
-   Data protection requirements for Workflow C should be identified during Discovery and included in the legal agreement planning.

**Develop**:

-   The supervisor workflows should be built in parallel with the worker agent, not after. The callback mechanism is a fundamental workflow design decision that cannot be retrofitted cleanly.
-   The shared state store should be provisioned as part of the BAIP environment setup. The one-time **Cloud ALM LMS `serviceId` registration** that enables the Workflow B / Workflow C telemetry pull path is a prerequisite of the same step.
-   During Internal Testing, the supervisor's behavior should be tested explicitly: does it correctly block invalid outputs, does it escalate via Task Center, does Workflow B fire anomalies on injected test data?

**Productization**:

-   The Sirius tasks for security validation include DPCE threat modelling and security concept documentation. The supervisor architecture — its agent identities, authorization policies, data flows, and sanitization rules — must be included in the security concept.
-   The production handover includes threshold configuration as a documented parameter set.
-   The supervisor's own telemetry and the shared baseline state require the same backup, recovery, and support model as the worker agent.


<a name="loio67417bb9efc147b1b8e722328394ebb0__section_summary"/>

## Summary

**Table: Key Principles**

<table>
<tr>
<th valign="top">

Aspect

</th>
<th valign="top">

Key Principle

</th>
</tr>
<tr>
<td valign="top">

Architectural position

</td>
<td valign="top">

SAP's Supervision tier — between platform governance and individual agent transactions

</td>
</tr>
<tr>
<td valign="top">

Three functions

</td>
<td valign="top">

Inline quality relay (Workflow A), telemetry monitor (Workflow B), support relay (Workflow C)

</td>
</tr>
<tr>
<td valign="top">

Worker-supervisor communication

</td>
<td valign="top">

sapAgent non-blocking + Webhook callback; sessionId for correlation

</td>
</tr>
<tr>
<td valign="top">

Verification stages

</td>
<td valign="top">

Deterministic first (if available), LLM critic second; disagreement → escalate

</td>
</tr>
<tr>
<td valign="top">

Data protection

</td>
<td valign="top">

Business data and reasoning stay in tenant; agent telemetry is stored in SAP Cloud ALM (SAP-operated, region-bound); sanitized extracts only leave via Workflow C on confirmed anomalies

</td>
</tr>
<tr>
<td valign="top">

Anomaly classifier

</td>
<td valign="top">

Rule-based only — no LLM on raw trace data

</td>
</tr>
<tr>
<td valign="top">

Fail-safe default

</td>
<td valign="top">

If supervisor unavailable, worker routes all outputs to sapTaskCenter

</td>
</tr>
<tr>
<td valign="top">

Key limitation

</td>
<td valign="top">

Latency on critical path; LLM critic quality requires its own test suite

</td>
</tr>
<tr>
<td valign="top">

FDE integration

</td>
<td valign="top">

Supervisor scope decided in Discovery; built in parallel with worker; included in security concept

</td>
</tr>
</table>

**Related Information**  


[Agent Quality Framework](agent-quality-framework-5f15077.md)

[Precision Guardrails](precision-guardrails-ae75c89.md)

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

[Reusable Components](reusable-components-ff99fe9.md)

[Proof of Concept Plan](proof-of-concept-plan-d8c5407.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio — low-code build environment for SAP AI agents, workflows, and tools.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right:
