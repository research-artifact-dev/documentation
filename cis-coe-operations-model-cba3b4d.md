<!-- loiocba3b4df8eb645849a65dfbedc58e880 -->

# CIS CoE Operations Model

The CIS Center of Excellence (CoE) Operations Model defines the lifecycle management layer that monitors deployed Industry AI agents, detects quality degradation early, intervenes when needed, and feeds operational findings back into the product. Use this model to transition from reactive incident response to proactive continuous lifecycle management across all FDE engagements.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_lifecycle"/>

## From Build-and-Hand-Over to Continuous Lifecycle Management

Traditional software delivery ends at go-live. The FDE model ends at productization — the agent becomes a standard SAP product, the customer signs a Cloud Contract, and the FDE team moves to the next engagement. The assumption embedded in this model is that a productized agent is a stable artifact: it continues to work correctly without active management.

AI agents do not behave this way. Their quality is a function of the runtime conditions they operate in — the data they receive, the models they use, the processes they support — and all of these change over time. A batch release agent that performed well during Early Adopter Care (EAC) may drift as the customer's process evolves. A pricing agent calibrated for one market may degrade when pricing dynamics shift. A maintenance agent may encounter equipment configurations it was never trained on.

This means the FDE model needs a lifecycle management layer that operates after productization: an operational capability that monitors deployed agents, detects degradation early, intervenes when needed, and feeds findings back into the product. CIS (Customer Industry Solutions) is the central, customer-anchored Industry AI acceleration engine — bringing together customer-facing delivery capabilities and customer-embedded Regional Engineering Pods to deliver Industry AI at scale.

**The core shift: from reactive to proactive.**

**Table: Traditional vs. CoE Operations**

<table>
<tr>
<th valign="top">

Traditional FDE Model

</th>
<th valign="top">

CIS CoE Operations Model

</th>
</tr>
<tr>
<td valign="top">

CIS engages when customer reports a problem

</td>
<td valign="top">

CIS is notified by the agent before the customer notices

</td>
</tr>
<tr>
<td valign="top">

Investigation requires access negotiation

</td>
<td valign="top">

The supervisor agent has already collected and sanitized the relevant evidence

</td>
</tr>
<tr>
<td valign="top">

Fix requires a new engagement

</td>
<td valign="top">

Many issues are resolved through configuration or minor workflow adjustments

</td>
</tr>
<tr>
<td valign="top">

Each engagement is independent

</td>
<td valign="top">

Findings across engagements feed back into improved patterns and productization

</td>
</tr>
</table>

The CIS CoE operations model does not replace anything that SAP's platform already provides. The BAIP platform harness, Joule Studio governance, and [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: observability remain the foundation. The CoE model adds the CIS-side operational layer on top: who is watching, what they do with what they see, and how they improve the agent over time.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_topologies"/>

## Two Deployment Topologies

The operational model differs depending on where the agent runs. Both topologies are valid Industry AI deployment scenarios.

### Topology A: Agent on Customer Tenant

The agent runs in the customer's own BAIP sub-account. The customer owns the runtime, the data, and the observability infrastructure. CIS has no default access.

**What CIS can observe without customer access:**

-   Automated support tickets created by Supervisor Workflow C (see [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)) — sanitized trace extracts, anomaly evidence, no business data
-   Process KPI reports shared by the customer as part of the engagement's value realization tracking

**Customer-centric observability (when CALM is deployed)**: when the customer has SAP Cloud ALM provisioned and configured for AI agent telemetry ingestion, the agent's operational metrics can be surfaced directly to the customer and their Customer Success Manager (CSM) via CALM. This construct — customer-centric observability — is a separate view from CIS-side monitoring: it makes agent health and performance metrics visible to the customer without requiring CIS access to their tenant. Confirm CALM availability and telemetry ingestion configuration in FDE Discovery. Note that CALM usage cannot be mandated at customer sites; this capability is available only when the customer has CALM deployed and agrees to configure it for the engagement.

**What requires customer consent for CIS access:**

-   Direct investigation in Agent Hub observability
-   Access to SAP Cloud ALM for specific incident analysis
-   Review of the audit trail for numerical decision quality (see [Precision Guardrails](precision-guardrails-ae75c89.md))

**The consent model**: CIS access to a customer tenant should be time-limited, scoped to the specific incident, and governed by the legal agreements already in place (Cloud Contract + SAP AI Terms for production). A standing Collaboration Agreement (CA) is typically not required if access is limited to anonymized operational data.

### Topology B: Agent on SAP Managed Tenant

The agent runs in a SAP-managed BAIP environment, typically as part of a SAP-hosted Industry AI subscription. SAP/CIS is the operator; the customer consumes the agent as a service.

**What CIS can observe directly:**

-   Full Agent Hub observability across all agents in the managed tenant
-   SAP Cloud ALM access without customer consent (it is SAP's infrastructure)
-   Audit trail data (subject to data processing agreements with the customer)
-   The full Automation tier: policy audits, identity lifecycle, risk assessment

**CIS operational responsibility is higher in this topology** — because SAP operates the BAIP environment, CIS carries responsibility for the platform and the agents running on it.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_lightspeed"/>

## Project LightSpeed — CIS Standard FDE Delivery Toolchain

Project LightSpeed (`github.tools.sap/CIS-Solution-Acceleration/Project-Lightspeed`) is the CIS-built delivery toolchain for FDE engagements, built by the CIS Solution Acceleration team. It is a 13-stage Claude Code plugin pipeline that transforms raw customer discovery artifacts — workshop notes, CDS models, OData specs, Z-table exports, meeting transcripts — into a deployed, observable, Joule-connected multi-agent application. The design ambition: one day from workshop to deployable solution design.

### Two-Phase Delivery Model

Project LightSpeed implements a two-phase delivery model that maps directly to FDE sprint milestones:

**Table: Project LightSpeed Phases**

<table>
<tr>
<th valign="top">

Phase

</th>
<th valign="top">

Scope

</th>
<th valign="top">

Target Timeline

</th>
<th valign="top">

Output

</th>
</tr>
<tr>
<td valign="top">

Phase 1 — Solution Design

</td>
<td valign="top">

Stages 1–8: ingest discovery artifacts, extract knowledge, ground to SAP canonical objects, run readiness gates, generate agent topology, compile `context-transfer.yaml`

</td>
<td valign="top">

Within one day of the customer workshop

</td>
<td valign="top">

Compiled knowledge artifact + agent topology design + readiness scores + coaching questions for any gaps

</td>
</tr>
<tr>
<td valign="top">

Phase 2 — Enterprise Integration & Production PoC

</td>
<td valign="top">

Stages 9–13: spec generation, LangGraph agent scaffolding, Joule capability registration, evaluation scorecard, BTP provisioning

</td>
<td valign="top">

Weeks (across the FDE Develop sprint)

</td>
<td valign="top">

Deployed agents with Joule surface, OTel observability, BTP infrastructure

</td>
</tr>
</table>

### Parallel-Terminal Workflow

LightSpeed uses an efficiency pattern the CoE should standardize across engagements:

-   **Terminal 1** — runs the discovery pipeline (Stages 1–8) as soon as workshop artifacts are available; produces the compiled `context-transfer.yaml`
-   **Terminal 2** — runs BTP provisioning in parallel via `btp-pilot` and the `pilot.yaml` desired-state file, so the deployment environment is ready when the build pipeline completes

Running both terminals in parallel eliminates the wait between "design complete" and "environment ready to deploy into."

### The Obsidian Vault as Live Architecture Knowledge Base

LightSpeed populates an Obsidian vault as each pipeline stage runs. By the time the pipeline completes, the vault contains a structured, auto-generated architecture knowledge base: extracted domain primitives, EKX-grounded entity mappings, agent topology, HITL gate locations, integration bindings, and readiness gate results. This vault is the customer workshop tool: it gives the FDE team and the customer a shared, inspectable artifact that reflects what was learned from the discovery data — not a slide deck authored offline.

### `pilot.yaml` — Engagement State Management for BTP

BTP provisioning is governed by a `pilot.yaml` desired-state file that declares the full target state of the customer's BTP sub-account. Key patterns:

-   **Park**: suspend an engagement's sub-account between sessions (reduces cost; state is preserved)
-   **Resume**: restore from parked state to active
-   **Adopt**: take ownership of an existing BTP sub-account into the LightSpeed provisioning model

The `pilot.yaml` file is the single engagement configuration artifact for infrastructure state. A `pilot.yaml.example` template is provided in the LightSpeed repository and is listed in the CIS component catalog (see [Reusable Components](reusable-components-ff99fe9.md)).

### Claude Code via Hyperspace Proxy

LightSpeed uses Claude Code as its execution engine, routed through SAP's Hyperspace proxy. No separate API keys are needed — engineers access Claude models through their existing SAP Hyperspace credentials. This makes LightSpeed immediately usable by any CIS engineer with Hyperspace access, with no additional provisioning.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_cis_proxy"/>

## The Supervisor as Autonomous CIS Proxy

On a customer tenant (Topology A), CIS cannot monitor continuously without permanent access. The supervisor agent resolves this constraint by acting as an autonomous CIS proxy — it does what CIS would do if it had access, and only escalates to CIS when it cannot resolve the issue itself.

**What the supervisor handles autonomously (without CIS involvement):**

**Table: Autonomous Supervisor Functions**

<table>
<tr>
<th valign="top">

Supervisor Function

</th>
<th valign="top">

How It Handles Autonomously

</th>
</tr>
<tr>
<td valign="top">

Individual output quality (Workflow A)

</td>
<td valign="top">

Verifies, approves, or blocks write-back; escalates to customer process owner via Task Center

</td>
</tr>
<tr>
<td valign="top">

Telemetry anomaly detection (Workflow B)

</td>
<td valign="top">

Classifies anomalies; alerts customer IT admin for non-critical issues

</td>
</tr>
<tr>
<td valign="top">

SAP support relay (Workflow C)

</td>
<td valign="top">

Creates sanitized support ticket; notifies customer IT admin

</td>
</tr>
</table>

**What escalates to CIS:**

-   Anomalies that Workflow B classifies as requiring SAP engineering-level investigation
-   Systematic quality degradation visible only across multiple invocations or over time
-   Issues that a support ticket alone cannot resolve — requiring the CIS team to review agent configuration, workflow logic, or model behavior

**The escalation path from supervisor to CIS:**

```
Supervisor Workflow B: detects systematic pattern across telemetry
         ↓
Workflow C: creates SAP support ticket (sanitized evidence)
         ↓
SAP support triages → assigns to CIS T&O team
         ↓
CIS T&O: determines whether customer consent for direct access is needed
    ├── No consent needed (sanitized evidence is sufficient to diagnose)
    │       → CIS proposes fix; customer approves; CIS or customer deploys
    └── Consent needed (deeper access required)
            → CIS initiates consent request to customer IT admin
              (time-limited, scoped access to specific agent/trace)
            → Investigation → fix → consent period ends
```

This design keeps CIS access exception-based rather than permanent. Permanent background access to customer tenants is not required — and is not appropriate under most legal frameworks.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_operating_rhythm"/>

## The CIS Operating Rhythm

For escalated issues that reach the CIS team, a defined operating rhythm ensures consistent, predictable response.

### Severity Tiers and Response Targets

**Table: Severity Tiers**

<table>
<tr>
<th valign="top">

Severity

</th>
<th valign="top">

Definition

</th>
<th valign="top">

CIS Response Target

</th>
</tr>
<tr>
<td valign="top">

Critical

</td>
<td valign="top">

Agent producing wrong numerical outputs that have been committed to system of record; regulatory impact possible

</td>
<td valign="top">

Same business day

</td>
</tr>
<tr>
<td valign="top">

High

</td>
<td valign="top">

Agent producing outputs that humans consistently override; process KPI degradation > 20% from baseline

</td>
<td valign="top">

Within 2 business days

</td>
</tr>
<tr>
<td valign="top">

Medium

</td>
<td valign="top">

Telemetry anomalies without confirmed business impact; elevated error rate

</td>
<td valign="top">

Within 5 business days

</td>
</tr>
<tr>
<td valign="top">

Low

</td>
<td valign="top">

Advisory: human override rate trending up; model drift detected early

</td>
<td valign="top">

Weekly review cycle

</td>
</tr>
</table>

These are proposed targets; the actual SLA depends on the customer contract and the Industry AI subscription terms.

**EAC support model**: during the Early Adopter Care phase, support follows a two-phase model. Phase 1 is guided, high-touch support provided directly by the development team (P&E). Phase 2 begins when ticket volume and active customer count exceed a threshold — CS&D takes the lead while P&E remains available for deep technical issues. The CIS CoE operating rhythm applies from Phase 2 onward.

### What CIS Can Do Without Redeployment

Many issues can be resolved without triggering a new FDE sprint:

-   **Threshold adjustment**: modify human approval thresholds (see [Precision Guardrails](precision-guardrails-ae75c89.md)) via the n8n Variables UI (`$vars`) — no workflow redeployment needed
-   **Prompt adjustment**: a prompt change requires a workflow redeployment — a lightweight operation, not a new FDE sprint. (The SAP Prompt Registry — versioned, governed, deployable *without* a workflow change — is a future capability, available when `promptTemplate` ships.)
-   **Supervisor rule update**: adjust Workflow B's anomaly classification thresholds in the shared state store or BAIP configuration
-   **Model swap**: change the model selection in the `sapOrchestration` node. A model swap requires editing and redeploying the workflow — an in-place fix, not a new FDE sprint.

### When to Trigger a New FDE Sprint

Some issues cannot be fixed in-place:

-   **Fundamental workflow logic error**: the agent's reason-act-observe loop has a structural flaw that requires workflow redesign
-   **Data quality issue**: the agent is receiving incorrect or incomplete data from the SAP system; requires upstream data fix or RAG corpus update
-   **New use case or regulatory requirement**: the agent's scope needs to expand beyond what was productized
-   **Persistent model drift**: the agent's quality has degraded to a point where re-evaluation with new ground-truth data and updated evals is required

These trigger a Microdelivery (incremental feature in existing product) or a new FDE engagement, depending on scope.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_improvement_loop"/>

## The Continuous Improvement Loop

The value of the CIS CoE model is not just in fixing issues — it is in making agents systematically better over time. This requires closing the loop between operational findings and build-time quality.

```
[Production runtime]
  Agent generates outputs → supervisor verifies → audit trail records results
         ↓
[Operational analysis (CIS T&O)]
  Review audit trail: verification rejection rates, human override patterns,
  confidence distributions, deviation from reference values,
  per-step tool call retry counts, self-correction loop depths, token consumption baselines
         ↓
[Categorize the finding]
    ├── Threshold issue → adjust configuration (in-place fix)
    ├── Prompt quality issue → update inline prompt + redeploy workflow (in-place fix)
    ├── Eval coverage gap → new test cases added to eval suite (build improvement)
    ├── Ground-truth data update → retrain or fine-tune model (productization item)
    ├── Iteration pattern on specific step → see iteration pattern analysis below
    └── Design flaw → Microdelivery or new FDE sprint
         ↓
[Build-time improvement]
  Updated evals + improved prompts + new ground-truth data →
  next agent version performs better on the cases that failed in production
         ↓
[Productization]
  Improvement becomes part of the standard product (not a customer-specific patch)
  → all customers on the same agent version benefit
```

**Topology dependency — CIS access to operational data**: the improvement loop requires CIS to review audit trail data and operational quality metrics. In Topology B (SAP managed tenant), CIS has direct access. In Topology A (agent on customer tenant), the audit trail and Agent Hub observability require customer consent for CIS access. The consent mechanism — scope, duration, and data sharing terms — should be established during FDE Discovery and included in the engagement's operational scope agreement.

### Iteration Pattern Analysis: Recognizing Optimization Signals

An agent that consistently retries a tool call, cycles through multiple self-correction loops, or consumes disproportionate tokens on a single step is exhibiting the same signal as a chatbot user who must rephrase the same request several times: a design mismatch between the task and the capability being used.

**Table: Signals to Monitor Per Step**

<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

What It Indicates

</th>
<th valign="top">

Optimization Candidate

</th>
</tr>
<tr>
<td valign="top">

Tool call retry count > 1 on same tool

</td>
<td valign="top">

Wrong parameters, insufficient input context, or poor tool fit

</td>
<td valign="top">

Prompt refinement; pre-call input validation; tool substitution

</td>
</tr>
<tr>
<td valign="top">

Self-correction loop count > 1 before pass or escalation

</td>
<td valign="top">

Agent's first output consistently fails its own verification

</td>
<td valign="top">

Verification threshold recalibration; prompt restructuring for first-time validity

</td>
</tr>
<tr>
<td valign="top">

Token consumption per step significantly above step baseline

</td>
<td valign="top">

LLM reasoning around missing context or ambiguous instructions

</td>
<td valign="top">

Context enrichment (RAG, richer system prompt); or route to a deterministic SAP service

</td>
</tr>
<tr>
<td valign="top">

Step latency outliers on specific input patterns

</td>
<td valign="top">

Disproportionate processing time for a subset of inputs

</td>
<td valign="top">

Identify triggering input characteristics; add pre-processing or fast-path routing

</td>
</tr>
<tr>
<td valign="top">

High LLM critic REJECT rate on specific output types

</td>
<td valign="top">

Agent consistently fails plausibility on a narrow output category

</td>
<td valign="top">

Few-shot examples for that output type; tighter Layer 1 input constraints

</td>
</tr>
</table>

The value of iteration analysis is in recognizing **consistent patterns across many invocations** — these patterns are invisible in a single audit trail entry; they surface only through aggregation.

**Table: Iteration Pattern to Action Mapping**

<table>
<tr>
<th valign="top">

Root Cause Identified

</th>
<th valign="top">

Category

</th>
<th valign="top">

Action

</th>
</tr>
<tr>
<td valign="top">

Agent lacks context to construct tool call correctly

</td>
<td valign="top">

Prompt quality issue

</td>
<td valign="top">

Update system prompt or RAG corpus — in-place fix

</td>
</tr>
<tr>
<td valign="top">

LLM computing what a deterministic SAP service should compute

</td>
<td valign="top">

Design issue

</td>
<td valign="top">

Replace with `sapMcpClient` call to authoritative service — Microdelivery

</td>
</tr>
<tr>
<td valign="top">

Verification threshold calibrated too tightly

</td>
<td valign="top">

Threshold issue

</td>
<td valign="top">

Recalibrate threshold — in-place fix

</td>
</tr>
<tr>
<td valign="top">

Input patterns not covered by build-time evals

</td>
<td valign="top">

Eval coverage gap

</td>
<td valign="top">

Add test cases for the triggering input patterns — build improvement

</td>
</tr>
<tr>
<td valign="top">

Task decomposition mismatch: step too complex for single agent

</td>
<td valign="top">

Design flaw

</td>
<td valign="top">

Redesign step or split into sub-agent — Microdelivery or new sprint

</td>
</tr>
</table>

### Agent Mining via Signavio Process Intelligence

> ### Note:  
> Agent mining availability applies to the next generation of SAP agents built on Joule Studio 2.0. For current FDE-delivered agents (SMJ orchestrator with a BYOA or Joule Studio Canvas runtime), Signavio agent mining is a forward-looking capability — not a current deliverable. The production monitoring approach for current agents is OpenTelemetry via the BAIP harness plus the supervisor telemetry patterns described in the [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md). Confirm Signavio availability in FDE Discovery before building operational dependencies on it.

Signavio Process Intelligence — accessible via the [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: Agent Mining capability — analyzes production agent execution traces to surface process-level behavioral patterns that are not visible in individual trace records.

Agent mining is distinct from supervisor real-time monitoring and from online eval statistical trends:

**Table: Monitoring Mechanism Comparison**

<table>
<tr>
<th valign="top">

Mechanism

</th>
<th valign="top">

What It Sees

</th>
<th valign="top">

Cadence

</th>
</tr>
<tr>
<td valign="top">

Supervisor Workflow A+B

</td>
<td valign="top">

Individual invocation quality; real-time anomalies

</td>
<td valign="top">

Per invocation (real-time)

</td>
</tr>
<tr>
<td valign="top">

Online eval (aeval)

</td>
<td valign="top">

Statistical quality trends; success rate drift; LLM-as-judge scoring

</td>
<td valign="top">

Daily / weekly aggregation

</td>
</tr>
<tr>
<td valign="top">

Agent mining (Signavio)

</td>
<td valign="top">

Process-level behavioral patterns across many executions; flow variants; deviation from expected paths

</td>
<td valign="top">

Periodic analysis (quarterly review cycle)

</td>
</tr>
</table>

Incorporate Signavio agent mining findings into the quarterly improvement review alongside iteration pattern analysis and online eval trends.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_maturity"/>

## CIS CoE Maturity Levels

The CoE operational capability can be assessed against four maturity levels. Most FDE engagements start at Level 1 and are designed to reach Level 2 at production handover; Levels 3 and 4 are organizational capabilities that develop over time.

**Table: CoE Maturity Levels**

<table>
<tr>
<th valign="top">

Level

</th>
<th valign="top">

Name

</th>
<th valign="top">

What It Looks Like

</th>
</tr>
<tr>
<td valign="top">

1 — Reactive

</td>
<td valign="top">

Incident response

</td>
<td valign="top">

CIS engages when the customer reports a problem; no automated monitoring; investigation starts from scratch

</td>
</tr>
<tr>
<td valign="top">

2 — Monitored

</td>
<td valign="top">

Automated alerting

</td>
<td valign="top">

Supervisor Workflows A+B are deployed; CIS is notified by automated tickets (Workflow C); response is faster because evidence is already collected

</td>
</tr>
<tr>
<td valign="top">

3 — Supervised

</td>
<td valign="top">

Cross-engagement visibility

</td>
<td valign="top">

CIS T&O has aggregated visibility across multiple deployed agents; can detect patterns that individual supervisors don't see; proactive recommendations before issues escalate

</td>
</tr>
<tr>
<td valign="top">

4 — Optimized

</td>
<td valign="top">

Continuous improvement

</td>
<td valign="top">

Production data systematically feeds build-time evals; prompts and models improve between versions; new engagements benefit from operational learnings of prior ones

</td>
</tr>
</table>

**Target state for FDE productization:** Level 2 is the minimum viable handover. The supervisor's three workflows should be deployed, tested, and operational before the production handover is complete. Handing over without the supervisor is handing over without the CIS proxy — the customer is on their own until something goes visibly wrong.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_fde_delivery"/>

## FDE Delivery Implications

The CIS CoE operations model has implications for how FDE engagements are scoped and delivered — it does not begin at productization, it is designed in Discovery.

**FDE process structure (as of Sep/Oct 2026)**: the FDE process has four phases: Nominate → Discover → Develop → Productize. Discovery runs for approximately **6 weeks** (weeks 1–6, with the FDE Gate at week 5). Productization starts at **RTC EAC** (first go-live with the customer's Early Adopter Care contacts), not at sprint end.

**During Discovery (~6 weeks)**:

-   Determine deployment topology: customer tenant or SAP managed tenant? This determines what CIS can observe without consent and shapes the operational model.
-   Identify the post-handover operational owner on the customer side and confirm their availability and responsibilities.
-   If CIS T&O operations are in scope: define the consent model for CIS access, the SLA tiers, and how Workflow C support tickets route to the T&O team.
-   Determine whether the customer's contract includes operational support by CIS T&O.

**During Develop**:

-   Supervisor Workflows A, B, and C should be built for operations, not just for the EAC phase. Workflow C in particular must be tested end-to-end.
-   Define the threshold configuration ownership: who can adjust thresholds post-handover — the customer's IT admin, the CIS T&O team, or both?
-   Build the operating rhythm documentation into the handover package. The T&O team should receive a briefing, not just a technical handover.

**During Productization**:

-   The production handover package must include the CIS T&O operational runbook: what the supervisor monitors, what each alert means, what CIS can fix in-place vs. what requires a new sprint.
-   The continuous improvement loop should be formally established: scheduled quarterly review of audit trail data, human override rates, and verification rejection rates as operational quality metrics.
-   **AI Agent Hub activation**: after deployment, newly registered agents appear in the AI Agent Hub in a **"not validated"** state until the customer IT admin explicitly verifies and activates them. Include this as a productization checklist item.
-   **LeanIX registration**: if the customer uses [AI Governance in SAP LeanIX](https://help.sap.com/docs/LEANIX/72d375467c1e4dcb872dfa2998b6328d/275887e57a441014876a9243fb67f92b.html "AI Governance in SAP LeanIX — enterprise architecture management, application inventory, and governance.") :arrow_upper_right: for enterprise architecture management, register the agent as an application component — owner, business domain, integration dependencies, and deprecation classification. LeanIX is used alongside the SAP AI Agent Hub: the Hub governs operational lifecycle; LeanIX provides the enterprise inventory and EA governance view.

**Delivery Path Guidance — Updated November 2026**

The delivery model is best understood as **two orthogonal decisions**:

-   **Axis 1 — orchestrator / Joule activation**: **SAP-Managed Joule (SMJ, "THE BOX") is the default and the only activation path that receives new features** (Joule Work, Joule Studio 2.0, new LoB onboarding). SMJ has been **GA since 2026-06-01**. **CMJ is legacy/deprecated** — a migration target, not a delivery choice for new engagements.
-   **Axis 2 — custom-agent runtime**: either **BYOA** (custom agent code on the customer's BTP subaccount — LangGraph/AG2/CAP/CrewAI — the standard, productizable runtime, registering to SMJ via A2A) **or Joule Studio 2.0** (low-code, SAP-managed). **BYOA is not a fallback**; it is the standard runtime for productization-intent FDE agents.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_adoption"/>

## Adoption Strategy — A Required FDE Handover Artifact

The FDE Engagement Playbook (Oct 2026) formalizes the **Adoption Strategy** as a mandatory, living document produced during every FDE engagement and handed over as part of the productization package — a structured document defining how the deployed agent will be adopted, sustained, and governed by the customer organization after the FDE team departs.

The [Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md) topic is authoritative for the Adoption Strategy — its seven-section structure, its production lifecycle across the engagement, and its quality gate. The nine-role governance roster below is maintained here as the CIS CoE's canonical governance reference (required by Adoption Strategy Section 3).

### Nine Adoption Governance Roles

**Table: Adoption Governance Roles**

<table>
<tr>
<th valign="top">

Role

</th>
<th valign="top">

Primary Responsibility

</th>
</tr>
<tr>
<td valign="top">

Executive Sponsor

</td>
<td valign="top">

Business commitment and escalation authority; champions the agent at the executive level; unblocks organizational resistance

</td>
</tr>
<tr>
<td valign="top">

Adoption Lead

</td>
<td valign="top">

Owns the Adoption Strategy document; orchestrates all workstreams; accountable for adoption metrics

</td>
</tr>
<tr>
<td valign="top">

Business Lead

</td>
<td valign="top">

Process owner; defines what good looks like; owns Dimension 2 process KPIs; approves threshold values

</td>
</tr>
<tr>
<td valign="top">

Customer IT Lead

</td>
<td valign="top">

Technical operations post-handover; owns BAIP application registrations and threshold configuration

</td>
</tr>
<tr>
<td valign="top">

Account Executive

</td>
<td valign="top">

Commercial relationship; ensures alignment between delivery and contract commitments

</td>
</tr>
<tr>
<td valign="top">

Enterprise Architect

</td>
<td valign="top">

Ensures architectural fit with customer's broader SAP landscape; reviews authorization matrix

</td>
</tr>
<tr>
<td valign="top">

Discovery / Execution Lead

</td>
<td valign="top">

FDE engagement lead; bridges discovery intent and build execution

</td>
</tr>
<tr>
<td valign="top">

Product / Solution Manager

</td>
<td valign="top">

SAP product management representation; ensures productization alignment

</td>
</tr>
<tr>
<td valign="top">

Test Lead

</td>
<td valign="top">

Owns QPA test strategy and UAT execution; acceptance sign-off before production go-live

</td>
</tr>
</table>

All nine roles must be named by end of Discovery. Unnamed roles at the FDE Gate are a Go/No-Go risk indicator — the engagement has no accountability structure for adoption.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_enable_track"/>

## The Enable Track — Customer Capability Transfer

Parallel to the **Build** track (agent construction) and **Operate** track (supervisor and monitoring setup), the FDE Engagement Playbook introduces the **Enable** track as a distinct, concurrent workstream: transferring the capability to operate and evolve the agent to the customer organization before the FDE team exits. The Enable track comprises a four-stage gradual release of responsibility (Demonstration → Shadowing → Reverse Shadowing → Independent Operation), role-based training for seven distinct audiences, and an Enable readiness checklist completed at productization handover. The Enable exit criterion is **Stage 3 minimum at handover; Stage 4 (independent operation) is the target**.

The [Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md) topic is authoritative for the Enable track detail: the four-stage model with per-stage evidence criteria and handover activities, the seven training audiences and their materials, and the Enablement Readiness Checklist.


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_cadence"/>

## Post-Engagement Cadence and Runbook Library

### Operating Rhythm Post-Handover

**Table: Post-Handover Operating Cadence**

<table>
<tr>
<th valign="top">

Cadence

</th>
<th valign="top">

Content

</th>
<th valign="top">

Participants

</th>
</tr>
<tr>
<td valign="top">

Weekly (hypercare period: first 30 days)

</td>
<td valign="top">

Review any Workflow B alerts; confirm threshold baselines are stable; triage any human override rate anomalies

</td>
<td valign="top">

CIS T&O + Customer IT Admin

</td>
</tr>
<tr>
<td valign="top">

Monthly

</td>
<td valign="top">

Review process KPI trends; review human override rate patterns; confirm eval suite is running in CI; review audit trail summary

</td>
<td valign="top">

CIS T&O + Business Lead

</td>
</tr>
<tr>
<td valign="top">

Quarterly

</td>
<td valign="top">

Full quality review: Dimension 1–3 metrics; agent mining (if Signavio available); improvement proposals; eval suite update; Adoption Strategy risk review

</td>
<td valign="top">

CIS T&O + Executive Sponsor + Business Lead

</td>
</tr>
</table>

**Hypercare period**: for the first 30 days post-go-live, CIS T&O operates at the weekly cadence regardless of issue volume. After 30 days, if the agent is stable (no Critical/High severity events, threshold baselines calibrated), cadence drops to monthly. The 90-day mark is the standard hypercare exit.

**30/60/90-day review milestones:**

**Table: Hypercare Milestones**

<table>
<tr>
<th valign="top">

Milestone

</th>
<th valign="top">

What Is Reviewed

</th>
</tr>
<tr>
<td valign="top">

30-day

</td>
<td valign="top">

Technical stability confirmed; threshold baselines calibrated; first human override rate data available; any blocking process alignment issues surfaced

</td>
</tr>
<tr>
<td valign="top">

60-day

</td>
<td valign="top">

Process KPI trend data emerging; user adoption metrics from Joule usage; any training gaps identified via support ticket analysis

</td>
</tr>
<tr>
<td valign="top">

90-day

</td>
<td valign="top">

Initial business value assessment against Adoption Strategy Section 2; Enable track Stage 4 confirmation; decision: extend hypercare or move to standard monthly cadence

</td>
</tr>
</table>

### Runbook Library (6 Operational Scenarios)

The T&O operational runbook (a required handover artifact) must include defined runbooks for the six most common operational scenarios:

**Table: Operational Runbooks**

<table>
<tr>
<th valign="top">

Scenario

</th>
<th valign="top">

First Action

</th>
<th valign="top">

Escalation Path

</th>
</tr>
<tr>
<td valign="top">

1. Workflow B alert: elevated human override rate

</td>
<td valign="top">

Check audit trail for the 24h window; identify which output type is being overridden; confirm it is not a data quality issue in the source system

</td>
<td valign="top">

If > 40% override rate for 3+ consecutive days: CIS T&O proposes threshold or prompt adjustment

</td>
</tr>
<tr>
<td valign="top">

2. Workflow B alert: token spike / runaway loop

</td>
<td valign="top">

Check Agent Hub for the specific invocation; confirm it is a runaway (not a legitimate large context); suspend the agent via Agent Hub Secure & Govern if confirmed

</td>
<td valign="top">

If confirmed runaway: execute rollback procedure (see [Production Operations](production-operations-e24808a.md)) while root cause is investigated

</td>
</tr>
<tr>
<td valign="top">

3. Quality regression post-update

</td>
<td valign="top">

Correlate regression signal with version deployment log; identify changed component; execute rollback in order: threshold → model → prompt → workflow

</td>
<td valign="top">

If rollback does not restore quality: trigger new FDE sprint scoped to the regression

</td>
</tr>
<tr>
<td valign="top">

4. Process KPI degradation (no version change)

</td>
<td valign="top">

Confirm the degradation in the business process layer (not just agent metrics); distinguish agent-caused from process-caused degradation

</td>
<td valign="top">

If agent-caused: review eval results for the affected test cases; update eval suite; may trigger prompt or model update

</td>
</tr>
<tr>
<td valign="top">

5. Customer requests a new capability

</td>
<td valign="top">

Assess against the continuous improvement framework: is this a configuration change, a prompt update, a Microdelivery, or a new FDE sprint?

</td>
<td valign="top">

Route through the appropriate channel: in-place fix (no sprint) or formal Microdelivery / new engagement request

</td>
</tr>
<tr>
<td valign="top">

6. Regulatory / audit requirement

</td>
<td valign="top">

Customer's compliance team requests evidence of agent decision-making for a specific transaction

</td>
<td valign="top">

Retrieve audit trail records for the correlation_id in question; provide: input snapshot, verification verdicts, human decision (if any), committed value, agent version. All required fields are in the standard audit trail schema (see [Precision Guardrails](precision-guardrails-ae75c89.md))

</td>
</tr>
</table>


<a name="loiocba3b4df8eb645849a65dfbedc58e880__section_summary"/>

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

Core shift

</td>
<td valign="top">

From reactive incident response to proactive continuous lifecycle management

</td>
</tr>
<tr>
<td valign="top">

Topology A (customer tenant)

</td>
<td valign="top">

CIS access is exception-based and consent-gated; supervisor acts as autonomous proxy; Workflow C initiates the access process when needed

</td>
</tr>
<tr>
<td valign="top">

Topology B (SAP managed tenant)

</td>
<td valign="top">

CIS is the operator; full Automation tier visibility; higher CIS operational responsibility

</td>
</tr>
<tr>
<td valign="top">

In-place fixes

</td>
<td valign="top">

Threshold adjustment (via `$vars`, no redeploy); prompt update (inline — requires a workflow redeploy, not a new sprint); model swap (workflow edit + redeploy); supervisor rule tuning

</td>
</tr>
<tr>
<td valign="top">

New sprint triggers

</td>
<td valign="top">

Structural workflow flaw, data quality issue, scope expansion, persistent model drift

</td>
</tr>
<tr>
<td valign="top">

Maturity target at handover

</td>
<td valign="top">

Level 2 (Monitored) minimum: supervisor deployed and operational before production handover is complete

</td>
</tr>
<tr>
<td valign="top">

FDE process (Sep 2026)

</td>
<td valign="top">

Nominate → Discover (~6 weeks) → Develop → Productize; productization starts at RTC EAC

</td>
</tr>
<tr>
<td valign="top">

FDE system of record

</td>
<td valign="top">

FDE on MXP (since Sep 2026); FDE Tracker (SharePoint) retired; engagement IDs are F-numbers

</td>
</tr>
<tr>
<td valign="top">

FDE design implication

</td>
<td valign="top">

Operations model must be designed in Discovery, not added at productization

</td>
</tr>
</table>

**Related Information**  


[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

[Precision Guardrails](precision-guardrails-ae75c89.md)

[Reusable Components](reusable-components-ff99fe9.md)

[Production Operations](production-operations-e24808a.md)

[Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md)

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right:

[AI Governance in SAP LeanIX](https://help.sap.com/docs/LEANIX/72d375467c1e4dcb872dfa2998b6328d/275887e57a441014876a9243fb67f92b.html "AI Governance in SAP LeanIX — enterprise architecture management, application inventory, and governance for AI components.") :arrow_upper_right:

[Joule Work – Develop](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/9d86b3b7b9534915a0f542ea7ea9f839.html "Joule Work — Develop: Joule Studio in the Joule Work environment for building and managing AI agents.") :arrow_upper_right:
