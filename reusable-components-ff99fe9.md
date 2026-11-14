<!-- loioff99fe9cde6447dcb7391ffb3f2fc552 -->

# Reusable Components

The CIS Delivery Efficiency framework classifies every design component from the Industry AI FDE concept into three reusability layers and defines the **CIS Starter Kit** — a versioned set of reference implementations that every FDE team inherits as a starting point. Use this catalog to reduce infrastructure rebuild effort, accumulate quality across engagements, and focus sprint capacity on the work that is genuinely engagement-specific.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_why_reuse"/>

## Why Reusability Is a First-Class Design Concern

CIS operates across multiple Industry AI FDE engagements simultaneously, and the number grows as the Autonomous Enterprise vision scales. Without intentional reuse, each engagement rebuilds the same infrastructure from scratch: three supervisor workflows, an audit trail schema, environment migration checklists, a rollback procedure. The build effort is real; the testing effort is real; and a freshly assembled Workflow B has less reliability evidence behind it than one that has run in production across four previous engagements.

Reuse is not just about efficiency. It is also about quality accumulation: production incidents discovered in one engagement feed back into the shared reference implementation so that every subsequent engagement benefits. This is the operational expression of the continuous improvement loop described in the [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md), applied not just to individual deployed agents but to the CIS delivery patterns themselves.

This topic maps every design component from the [Agent Quality Framework](agent-quality-framework-5f15077.md) through [Production Operations](production-operations-e24808a.md) to its reusability profile: what stays the same across engagements, what requires per-engagement configuration, and what is genuinely engagement-specific.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_reusability_stack"/>

## The Reusability Stack

The concept components organize into three layers. The distinction matters because each layer has a different delivery model: the first is inherited, the second is configured, the third is built.

### Layer 1 — Deploy As-Is

Zero per-engagement content. These components are structurally and logically domain-agnostic: they operate on telemetry signals, schema definitions, process checklists, and design rules that do not change regardless of which Industry AI domain is being served or which worker agent is being supervised.

CIS T&O builds these once, maintains them as versioned reference artifacts, and provides them to FDE teams to inherit unchanged.

### Layer 2 — Parameterize and Deploy

Fixed skeleton; per-engagement content in defined slots. The workflow structure, routing logic, and data schemas are the same in every engagement. What varies are values (threshold calibrations, recipient IDs, connection credentials), prompts (LLM critic system prompts), and rules (the specific verification logic for this agent's output type).

CIS T&O maintains the skeleton. The FDE team fills the per-engagement slots during Discovery and Develop, and documents the resulting configuration in the production handover package.

### Layer 3 — Engagement-Specific

Cannot be lifted across engagements. This is the worker agent's business logic, the process KPIs that define what the agent is optimizing for, the verification rules specific to this output type, and the threshold values agreed with the process owner. These require genuine per-engagement design work; attempting to reuse them from a prior engagement introduces the wrong assumptions.

The total build scope of a new FDE engagement is the Layer 3 content plus the parameterization work for Layer 2. Layer 1 is not in scope for the FDE sprint.

> ### Note:  
> **Layer denotes reusability profile, not current availability.** A component's Layer is its reusability profile — deploy-as-is vs. parameterize vs. engagement-specific — which is independent of whether a forkable reference artifact exists today. Several Layer 1/2 reference artifacts — Supervisor Workflows B and C, the shared-state-store DDL, and the APM archetype config — are **design targets** not yet published as standalone, forkable artifacts. The authoritative statement of what is available now is the CIS Starter Kit status table below.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_component_analysis"/>

## Component-by-Component Analysis

### Layer 1 — Deploy As-Is

**Table: Layer 1 Components**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Topic Reference

</th>
<th valign="top">

What Is Fixed

</th>
<th valign="top">

Parameterization Surface

</th>
<th valign="top">

Maintenance Owner

</th>
</tr>
<tr>
<td valign="top">

Supervisor Workflow C (Support Relay)

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Full workflow: sanitization rules, SAP Cloud ALM ITSM Adapter API call, IT admin notification

</td>
<td valign="top">

Customer tenant ID (from environment config, not hardcoded)

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Audit trail schema — base fields

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

All 16 fields: timestamp, agent_id, agent_version, correlation_id, input_state, agent_output, stage verdicts, human_decision, human_modified_value, committed_value, delta

</td>
<td valign="top">

Table name prefix per engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Audit trail schema — cost extension

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Four additional fields: input_tokens, output_tokens, total_tokens_invocation, model_used

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Telemetry anomaly rule set (Workflow B rule logic)

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

All six rule types: error rate, tool call count per invocation, response latency, human override rate, verification rejection rate, identical input pattern

</td>
<td valign="top">

None — rule types are domain-agnostic; threshold values are Layer 2

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Sanitization rules table (what to strip vs. retain in trace exports)

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Field-by-field retain/strip classification

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Quality signal escalation structure (Level 0–3)

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Four-level structure: automated → monitored → supervised → blocked; and the logic distinguishing each level

</td>
<td valign="top">

Specific threshold values and recipient IDs are Layer 2

</td>
<td valign="top">

CIS T&O (structure)

</td>
</tr>
<tr>
<td valign="top">

Version stack discipline (5 components)

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Definition of the five-component version stack; change classification table (Low / Medium / High / Very High risk)

</td>
<td valign="top">

Applied per engagement to that engagement's specific component versions

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Supervisor fail-safe rule

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

If supervisor unavailable, worker routes all outputs to sapTaskCenter — never to autonomous write-back

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Three things never to copy between environments

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Credentials, dev/test thresholds, agent identities — all must be re-provisioned per environment

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

When NOT to use multi-agent (decision table)

</td>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
<td valign="top">

Full decision table with six rows; criteria for single agent vs. multi-agent

</td>
<td valign="top">

None

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Three pre-migration checklists (dev→test, test→EAC, EAC→prod)

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

All checklist items across three transitions

</td>
<td valign="top">

None — items are the same for every engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Prompt injection defense system prompt template

</td>
<td valign="top">

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
<td valign="top">

Boilerplate instruction text

</td>
<td valign="top">

Role description at the opening line

</td>
<td valign="top">

FDE team

</td>
</tr>
</table>

### Layer 2 — Parameterize and Deploy

**Table: Layer 2 Components**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Topic Reference

</th>
<th valign="top">

Fixed Skeleton

</th>
<th valign="top">

Parameterization Surface

</th>
<th valign="top">

Maintenance Owner

</th>
</tr>
<tr>
<td valign="top">

Supervisor Workflow B (Telemetry Monitor) — full workflow

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Schedule trigger, HTTP Request to SAP Cloud ALM + AI Core usage API, Code node rule structure, routing logic, IT admin alert path, cost anomaly checks, Workflow C trigger

</td>
<td valign="top">

Anomaly threshold values (calibrated from EAC baseline); schedule interval; Task Center recipient IDs

</td>
<td valign="top">

CIS T&O (template); FDE team (calibration)

</td>
</tr>
<tr>
<td valign="top">

Supervisor Workflow A — routing skeleton

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Webhook trigger, If/Switch routing (BOTH CONFIRM / BOTH REJECT / DISAGREE / low confidence), sapAgent approval signal back to worker, Task Center escalation path, audit trail write

</td>
<td valign="top">

Code node verification rules (engagement-specific); LLM critic systemPrompt; approval threshold value

</td>
<td valign="top">

CIS T&O (skeleton); FDE team (Code node + critic prompt)

</td>
</tr>
<tr>
<td valign="top">

Supervisor shared state store schema

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
<td valign="top">

Baseline table structure (rolling metrics per agent); Pending Set table structure for multi-agent Topology 3

</td>
<td valign="top">

Table name prefix per engagement; initial state is always empty per environment

</td>
<td valign="top">

CIS T&O (DDL template)

</td>
</tr>
<tr>
<td valign="top">

Four-layer guardrail stack structure

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

Layer sequence (Pre-Check → Output Validation → Human Approval → Audit Trail); which layers are mandatory vs. optional per output risk level

</td>
<td valign="top">

Layer 2 reference calculation service; Layer 3 threshold values; Layer 1 constraint source

</td>
<td valign="top">

FDE team (configured with process owner for thresholds)

</td>
</tr>
<tr>
<td valign="top">

sapTaskCenter customAttributes pattern

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md), [Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
<td valign="top">

Six-attribute set: proposed value, reference value, deviation %, agent confidence, stage verdicts, correlation ID

</td>
<td valign="top">

Attribute display labels; recipient IDs; priority mapping

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Worker agent base pattern (n8n workflow template)

</td>
<td valign="top">

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
<td valign="top">

sapOrchestration + sapMcpClient + verification node structure; routing to supervised (sapAgent callback) or unsupervised (inline)

</td>
<td valign="top">

Agent task definition in systemPrompt; SAP system connections; specific MCP tools wired

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Three-dimension quality framework (evaluation design)

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Framework structure (Technical / Process KPI / Business Outcome); Dimension 1 metric types and their Goodhart's Law guard

</td>
<td valign="top">

Dimension 2 process KPIs (engagement-specific, defined in TO-BE process flow); Dimension 3 business outcome metrics

</td>
<td valign="top">

FDE team (with process owner for Dimensions 2–3)

</td>
</tr>
<tr>
<td valign="top">

Eval test suite structure

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Scenario type categories (high-freq/low-risk, low-freq/high-risk, exception, novel); acceptance threshold approach per category

</td>
<td valign="top">

Specific test cases; acceptance threshold values; test data strategy

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Rollback procedure template

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Trigger criteria (human override rate, rejection rate, KPI degradation, format failures); rollback order (threshold → model → prompt → workflow); state continuity rules (audit trail immutable, baseline reset)

</td>
<td valign="top">

Authorization path per topology; specific component versions being rolled back

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Severity tier table and operating rhythm

</td>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

</td>
<td valign="top">

Four severity tiers (Critical / High / Medium / Low) with response target structure

</td>
<td valign="top">

Actual SLA commitments (agreed per contract); T&O team contact details

</td>
<td valign="top">

CIS T&O (with contract alignment)

</td>
</tr>
<tr>
<td valign="top">

Continuous improvement loop

</td>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

</td>
<td valign="top">

Loop structure: production findings → categorize → in-place fix vs. build improvement vs. new sprint; iteration pattern analysis signal types

</td>
<td valign="top">

Finding categories and escalation paths for this engagement

</td>
<td valign="top">

CIS T&O

</td>
</tr>
</table>

### Layer 3 — Engagement-Specific

**Table: Layer 3 Components**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Topic Reference

</th>
<th valign="top">

Why It Cannot Be Reused

</th>
<th valign="top">

Who Authors It

</th>
</tr>
<tr>
<td valign="top">

Worker agent business logic

</td>
<td valign="top">

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
<td valign="top">

The agent's task, SAP system connections, tools, and goal state are unique to this use case and customer

</td>
<td valign="top">

FDE Application Engineer + Domain PM

</td>
</tr>
<tr>
<td valign="top">

Process KPIs (Dimension 2 of quality framework)

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Defined in the TO-BE process flow during Discovery; entirely domain- and customer-specific

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Workflow A Code node verification rules

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

Specific range checks, unit checks, tolerance values apply only to this agent's output type

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

LLM critic systemPrompt

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

What a "correct" output means is different for a batch release quantity vs. a pricing recommendation

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

Human approval threshold values

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

Business decision agreed with process owner in Discovery; not derivable from a template

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Authorization matrices (all agents)

</td>
<td valign="top">

[Security and Data Protection](security-and-data-protection-a403094.md), [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
<td valign="top">

Scoped to specific SAP systems and entities this engagement accesses; differs per customer and per worker

</td>
<td valign="top">

FDE team + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Anomaly detection threshold values

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Calibrated from EAC baseline observation specific to this agent's behavior; cannot be borrowed from another agent's deployment

</td>
<td valign="top">

CIS T&O (post-EAC calibration)

</td>
</tr>
<tr>
<td valign="top">

Regulatory audit trail retention period

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

Confirmed with customer's compliance team; jurisdiction- and industry-specific

</td>
<td valign="top">

Customer compliance team

</td>
</tr>
<tr>
<td valign="top">

Eval test cases and test data

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Test scenarios must cover this agent's boundary conditions and edge cases

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

LLM critic test suite

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

The critic prompt's quality is engagement-specific; its test suite must match its content

</td>
<td valign="top">

FDE team

</td>
</tr>
</table>


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_starter_kit"/>

## The CIS Starter Kit

The CIS Starter Kit is the versioned set of Layer 1 and Layer 2 artifacts that CIS T&O maintains and provides to every FDE engagement as a starting point. It is not a monolithic package — FDE teams take what applies to their engagement's scope and leave what does not.

**Table: Starter Kit Component Status**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Status

</th>
<th valign="top">

Available Where

</th>
</tr>
<tr>
<td valign="top">

Workflow A skeleton (`cis-supervisor-workflow-a-skeleton`)

</td>
<td valign="top">

PoC in progress

</td>
<td valign="top">

[Proof of Concept Plan](proof-of-concept-plan-d8c5407.md) — Workflow A PoC is the first build step

</td>
</tr>
<tr>
<td valign="top">

Workflow B reference (`cis-supervisor-workflow-b-ref`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

Not yet a standalone artifact; build per [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md) patterns

</td>
</tr>
<tr>
<td valign="top">

Workflow C reference (`cis-supervisor-workflow-c-ref`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

Not yet a standalone artifact; patterns from [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
</tr>
<tr>
<td valign="top">

Worker agent base (`cis-worker-agent-base`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

LightSpeed `references/supply-chain-risk-agent/` and `references/service-technician-agent/`; cookbook `templates/supply-chain-risk-agent/`

</td>
</tr>
<tr>
<td valign="top">

Audit trail DDL (`cis-audit-trail-ddl.sql`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

Schema per [Precision Guardrails](precision-guardrails-ae75c89.md) and [Production Operations](production-operations-e24808a.md); provisioned by the LightSpeed BTP Terraform modules (`infra/btp/`)

</td>
</tr>
<tr>
<td valign="top">

Shared state store DDL (`cis-shared-state-store-ddl.sql`)

</td>
<td valign="top">

Design target

</td>
<td valign="top">

DDL schema specified in [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md) and [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md); standalone file not yet published

</td>
</tr>
<tr>
<td valign="top">

Pre-migration checklists (`cis-pre-migration-checklists.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md) templates; copy and fill per engagement

</td>
</tr>
<tr>
<td valign="top">

Config naming convention (`cis-config-naming-convention.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md) — derive from documented pattern

</td>
</tr>
<tr>
<td valign="top">

Rollback runbook template (`cis-rollback-runbook-template.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md) — template with placeholders

</td>
</tr>
<tr>
<td valign="top">

Sanitization rules (`cis-sanitization-rules.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
</tr>
<tr>
<td valign="top">

Security checklist (`cis-security-checklist.md`)

</td>
<td valign="top">

Available now

</td>
<td valign="top">

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
</tr>
</table>

**Summary**: worker agent and schema artifacts are usable today via LightSpeed; the supervisor reference workflows (A, B, C) are the build priority. Teams forking LightSpeed components as an interim measure should document the divergence from the eventual Starter Kit version in their productization package.

### Starter Kit Contents

**Joule Studio Reference Workflows** (maintained in a CIS-internal [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio — low-code build environment for SAP AI agents, workflows, and tools.") :arrow_upper_right: project, version-controlled, tested against the current BAIP platform version):

**Table: Joule Studio Reference Workflows**

<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
<th valign="top">

Fork and Configure

</th>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-b-ref`

</td>
<td valign="top">

Workflow B (Telemetry Monitor) with full rule set and cost extension; no engagement-specific content — **design target, not yet a standalone forkable artifact**

</td>
<td valign="top">

When published: fork; set anomaly thresholds after EAC baseline; configure recipient IDs

</td>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-c-ref`

</td>
<td valign="top">

Workflow C (Support Relay) — **design target, not yet a standalone forkable artifact**

</td>
<td valign="top">

When published: fork; verify tenant ID injection from environment config

</td>
</tr>
<tr>
<td valign="top">

`cis-supervisor-workflow-a-skeleton`

</td>
<td valign="top">

Workflow A with routing structure, webhook trigger, sapAgent callback, Task Center escalation, audit trail write; Code node placeholder for verification rules

</td>
<td valign="top">

Fork; author the Code node verification rules and LLM critic systemPrompt for this engagement

</td>
</tr>
<tr>
<td valign="top">

`cis-worker-agent-base`

</td>
<td valign="top">

Worker agent base pattern with sapOrchestration, sapMcpClient, verification routing, and supervised/unsupervised path

</td>
<td valign="top">

Fork; define task systemPrompt, wire SAP system connections, add MCP tools

</td>
</tr>
</table>

**Workflow A scoping: one skeleton per output type, not per agent**

The Workflow A skeleton is forked per output type — not per worker agent and not per engagement. Two workers that produce structurally similar outputs can share one Workflow A instance: the Code node verification rules and the LLM critic system prompt are the same because the output type is the same; threshold values that differ between workers are looked up by `agent_identity` in the Code node rather than duplicated into separate workflows.

Sharing breaks down when workers produce fundamentally different output types — different units, different validation logic, different reference calculation services. The rule of thumb: if the Code node rules would be identical for two workers, share the workflow; if they diverge significantly, create a separate instance.

**Schema Artifacts** (DDL scripts):

**Table: Schema Artifacts**

<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

`cis-audit-trail-ddl.sql`

</td>
<td valign="top">

HANA Cloud DDL for the append-only audit trail table; includes all base fields and cost extension fields from [Precision Guardrails](precision-guardrails-ae75c89.md) and [Production Operations](production-operations-e24808a.md)

</td>
</tr>
<tr>
<td valign="top">

`cis-shared-state-store-ddl.sql`

</td>
<td valign="top">

**Design target:** HANA Cloud DDL for the supervisor shared state store (baseline table + Pending Set table for multi-agent Topology 3). The schema is specified in [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md) and [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md); a standalone DDL file is not yet published

</td>
</tr>
</table>

**HANA Cloud provisioning**: both DDL artifacts target HANA Cloud (not HANA on-premise). In Project LightSpeed, `pilot.yaml` handles HANA Cloud provisioning as part of the BTP desired-state model — `cookbookctl apply` provisions the HANA Cloud instance, schema, and all required tables in a single operation. For engagements not using LightSpeed, provision HANA Cloud manually via the BTP cockpit. Estimated storage: plan for 10–50 MB per month of production operation per agent; the append-only audit trail grows continuously — confirm a retention policy with the customer compliance team and implement scheduled archiving after 90 days.

**Process and Configuration Artifacts**:

**Table: Process and Configuration Artifacts**

<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

`cis-pre-migration-checklists.md`

</td>
<td valign="top">

Three environment transition checklists (dev→test, test→EAC, EAC→prod); used unchanged for every engagement

</td>
</tr>
<tr>
<td valign="top">

`cis-config-naming-convention.md`

</td>
<td valign="top">

Configuration parameter naming convention and environment-agnostic wiring pattern

</td>
</tr>
<tr>
<td valign="top">

`cis-rollback-runbook-template.md`

</td>
<td valign="top">

Rollback procedure template with placeholders for engagement-specific component versions and authorization contacts

</td>
</tr>
<tr>
<td valign="top">

`cis-sanitization-rules.md`

</td>
<td valign="top">

Sanitization rules table; included by reference in the Sirius security concept documentation

</td>
</tr>
<tr>
<td valign="top">

`cis-security-checklist.md`

</td>
<td valign="top">

Per-workflow security checklist; used as a review gate before every environment transition

</td>
</tr>
</table>

**Evaluation Skills** (Claude Code skills, installable via `npx skills add` or `agpm`):

**Table: Evaluation Skills**

<table>
<tr>
<th valign="top">

Skill

</th>
<th valign="top">

Description

</th>
<th valign="top">

Install Command

</th>
</tr>
<tr>
<td valign="top">

`agentic-eval`

</td>
<td valign="top">

11+1 Agentic Engineering Spec Quality Assessment; evaluates spec completeness, AI-readiness, code-spec alignment, MCP utilization, and SAP standards compliance. Run at end of Discovery and pre-EAC. HTML and Word report export.

</td>
<td valign="top">

`npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval`

</td>
</tr>
</table>

See [Agent Evaluation](agent-evaluation-bb08bc0.md) for the full criteria framework, score thresholds, and sprint timing guidance.

### Starter Kit Governance

The Starter Kit is owned by CIS T&O and governed as a product, not as documentation:

-   **Version**: each release of the Starter Kit is tagged. FDE teams record which Starter Kit version their engagement forked from.
-   **Update cadence**: the Starter Kit is updated when BAIP platform changes affect the reference workflows, when a production incident reveals a gap in the rule set or schema, or when a completed engagement produces a generalizable improvement.
-   **Improvement pipeline**: production findings from the continuous improvement loop that are generalizable across engagements are proposed as Starter Kit updates. CIS T&O reviews and merges them.
-   **Deprecation**: when BAIP node APIs or [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio — low-code build environment for SAP AI agents, workflows, and tools.") :arrow_upper_right: behavior changes, the Starter Kit is updated before the next engagement forks from it.

### Customer-Managed Skill Registry

Beyond the Starter Kit's cross-engagement reference artifacts, high-maturity CIS deployments benefit from a **customer-managed Skill Registry** — a governed store of reusable agent behaviors specific to that customer's SAP landscape:

**Table: Skill Registry Governance Features**

<table>
<tr>
<th valign="top">

Governance Feature

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

Version pinning

</td>
<td valign="top">

Skills are versioned; agents pin to a specific skill version rather than pulling the latest

</td>
</tr>
<tr>
<td valign="top">

Approval workflows

</td>
<td valign="top">

New or updated skills require review and sign-off before they can be called in production

</td>
</tr>
<tr>
<td valign="top">

Cross-team distribution

</td>
<td valign="top">

Skills developed for one agent are discoverable and reusable by other agents in the same tenant — turning project-local patterns into tenant-wide assets

</td>
</tr>
<tr>
<td valign="top">

Compounding knowledge

</td>
<td valign="top">

Every production fix, edge case, and workaround feeds back into the registry as an updated skill; operational learning accumulates rather than being lost

</td>
</tr>
</table>

**FDE delivery implication**: the Skill Registry is a productization and handover consideration. When the FDE team hands over to the customer's IT admin and CIS T&O, the registry is one of the governance artifacts the customer IT team should own and operate.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_lightspeed"/>

## Project LightSpeed: CIS Reference Implementation

Project LightSpeed (`github.tools.sap/CIS-Solution-Acceleration/Project-Lightspeed`) is the current, built CIS component library. Where the CIS Starter Kit describes a specification of what should be available, LightSpeed is what exists today. FDE teams should treat the LightSpeed component catalog as the authoritative starting point; Starter Kit artifacts that do not yet exist in a standalone CIS project can be bootstrapped from the corresponding LightSpeed component.

### Component Catalog

**Table: LightSpeed Component Catalog**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Location in Repo

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

OTel/logging middleware

</td>
<td valign="top">

`references/middleware/`

</td>
<td valign="top">

Drop-in Python package; 2-line setup; graceful degradation (Cloud Logging, env-OTLP, logging-only fallback); works in local dev, SAP Cloud Logging, and sovereign regions

</td>
</tr>
<tr>
<td valign="top">

Supply-chain risk reference agent

</td>
<td valign="top">

`references/supply-chain-risk-agent/`

</td>
<td valign="top">

Reference agent implementation with live S/4HANA tooling; starting point for supply-chain domain engagements

</td>
</tr>
<tr>
<td valign="top">

Service technician reference agent

</td>
<td valign="top">

`references/service-technician-agent/`

</td>
<td valign="top">

Reference agent implementation with field service tooling; starting point for asset management domain engagements

</td>
</tr>
<tr>
<td valign="top">

BTP Terraform modules

</td>
<td valign="top">

`infra/btp/`

</td>
<td valign="top">

Reusable Terraform modules: foundation stack, identity stack, joule stack; parameterized for BTP sub-account provisioning

</td>
</tr>
<tr>
<td valign="top">

S/4HANA entity catalog

</td>
<td valign="top">

`catalog/s4_catalog.sqlite`

</td>
<td valign="top">

Offline SQLite catalog of S/4HANA canonical objects, attributes, and APIs; enables entity grounding without an EKX connection at build time

</td>
</tr>
<tr>
<td valign="top">

A2A Agent Toolkit

</td>
<td valign="top">

`toolkits/a2a-agent-toolkit/`

</td>
<td valign="top">

Agent scaffolding toolkit in TypeScript (CAP + Express) and Python (FastAPI + LangGraph) implementing the A2A inter-agent protocol

</td>
</tr>
<tr>
<td valign="top">

Discovery/build skills

</td>
<td valign="top">

30+ skill files across pipeline stages

</td>
<td valign="top">

Reusable Claude Code skills covering all 13 pipeline stages; each skill is independently usable on other projects

</td>
</tr>
<tr>
<td valign="top">

Engagement config template

</td>
<td valign="top">

`pilot.yaml.example`

</td>
<td valign="top">

Template for the BTP desired-state file (park/resume/adopt patterns); copy and parameterize per engagement

</td>
</tr>
</table>

The OTel middleware, Terraform modules, and A2A toolkit are drop-in components that can be adopted into any CIS engagement regardless of whether the full LightSpeed pipeline is used.

**RAG ingest pipeline** (`fde_rag_ingest.py`): Project LightSpeed includes a standard RAG ingest component as a stage 9 output. This component handles document chunking, embedding generation, and vector store population for BTP-hosted vector stores. It is parameterized for the engagement's grounding corpus and is a reusable Layer 2 component — the pipeline structure is fixed; the corpus configuration is per-engagement. For any FDE agent that uses RAG grounding (`sapOrchestration groundingConfig`), the LightSpeed RAG ingest pipeline should be the starting point rather than a custom implementation.

### Versioned-Skill Distribution Pattern

The `book-of-fde` and LightSpeed both use the same distribution model for Claude Code skills: a skill is a Markdown file (`skill-name.md`) stored in a git repository. Installation is a single `gh api` call against `github.tools.sap`; contribution back is a PR.

**Table: Versioned-Skill Distribution Operations**

<table>
<tr>
<th valign="top">

Operation

</th>
<th valign="top">

Mechanism

</th>
</tr>
<tr>
<td valign="top">

Install a skill

</td>
<td valign="top">

`gh api` call against the repository; skill file written to `~/.claude/skills/`

</td>
</tr>
<tr>
<td valign="top">

Contribute a fix or improvement

</td>
<td valign="top">

PR against the source repository

</td>
</tr>
<tr>
<td valign="top">

Enforce version consistency

</td>
<td valign="top">

Pre-commit hook increments version on every change to the skill file

</td>
</tr>
<tr>
<td valign="top">

Pin a specific version

</td>
<td valign="top">

Reference the commit SHA or tag in the install command

</td>
</tr>
<tr>
<td valign="top">

Audit skill contents

</td>
<td valign="top">

Inspect the Markdown file directly — no binary packaging

</td>
</tr>
</table>

### Security Note: Skills Are Customer-Inspectable

Skill files are plain Markdown and are readable by anyone with access to the Claude session or the repository. This is a feature (auditability) and a constraint (confidentiality):

-   **Skills are appropriate for**: process automation, phase routing, checklist enforcement, tool orchestration, and coordination logic that can be shared with customers or disclosed in an audit.
-   **Skills are NOT appropriate for**: proprietary algorithms, confidential business rules, pricing logic, or any logic that must remain confidential relative to the customer.
-   **For IP-sensitive logic**: implement in a code-based agent (Python/TypeScript), not a skill.

### Custom Agentic Solutions CoE Cookbook — Second CIS Reference Library

Alongside Project LightSpeed, the **Custom Agentic Solutions CoE Cookbook** (`github.tools.sap/<internal-org>/Custom-Agentic-solutions-CoE`) is a second CIS reference implementation and field delivery resource. Where LightSpeed focuses on the end-to-end FDE pipeline (13 stages from discovery artifacts to deployed multi-agent application), the cookbook focuses on the agent delivery primitives: how to scaffold a BTP-hosted A2A agent, deploy it to CF or Kyma, and wire it into Joule.

**Table: Cookbook Structure**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Location in Repo

</th>
<th valign="top">

What It Provides

</th>
</tr>
<tr>
<td valign="top">

A2A Agent Toolkit

</td>
<td valign="top">

`toolkits/a2a-agent-toolkit/`

</td>
<td valign="top">

Claude Code plugin (`sap-a2a-agent-toolkit`, v0.6.0) that scaffolds and deploys A2A agents; drives Checkpoints 1–3

</td>
</tr>
<tr>
<td valign="top">

Supply-chain risk agent template

</td>
<td valign="top">

`templates/supply-chain-risk-agent/`

</td>
<td valign="top">

Canonical Python A2A agent starter (four-file shape: `agent.py`, `agent_executor.py`, `app.py`, `__main__.py`)

</td>
</tr>
<tr>
<td valign="top">

BTP deploy recipes

</td>
<td valign="top">

`recipes/02-deploy-btp/`

</td>
<td valign="top">

CF and Kyma deployment patterns with smoke tests

</td>
</tr>
<tr>
<td valign="top">

Sovereign model gateway

</td>
<td valign="top">

`recipes/optional/sovereign-model-gateway/`

</td>
<td valign="top">

Required for sovereign/regulated regions; OpenAI-compatible gateway as AI Core replacement

</td>
</tr>
<tr>
<td valign="top">

FDE code-first stack reference

</td>
<td valign="top">

`references/fde-code-first-stack.md`

</td>
<td valign="top">

North-star stack (Kyma + A2A v1.0 + SAP Cloud SDK for Python); ownership split; access checklist

</td>
</tr>
</table>

**Three-checkpoint delivery playbook:**

**Table: Cookbook Delivery Checkpoints**

<table>
<tr>
<th valign="top">

Checkpoint

</th>
<th valign="top">

Recipe

</th>
<th valign="top">

Output Evidence

</th>
</tr>
<tr>
<td valign="top">

1 — Local agent

</td>
<td valign="top">

`recipes/01-scaffold-agent/`

</td>
<td valign="top">

`SendMessage` returns non-error response locally

</td>
</tr>
<tr>
<td valign="top">

2 — BTP deploy

</td>
<td valign="top">

`recipes/02-deploy-btp/`

</td>
<td valign="top">

Agent Card reachable at public URL; smoke test passes

</td>
</tr>
<tr>
<td valign="top">

3 — Joule wiring

</td>
<td valign="top">

`recipes/03-joule/`

</td>
<td valign="top">

`joule list capabilities` shows ACTIVE; Joule routes a real prompt to the agent

</td>
</tr>
</table>


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_discovery_parameterization"/>

## What Parameterization Happens in FDE Discovery

Discovery is where Layer 2 skeletons acquire their per-engagement content. The following decisions must be made in Discovery — before the Develop phase begins — to ensure the FDE sprint can proceed without scope uncertainty:

**Table: Discovery Decisions and Owners**

<table>
<tr>
<th valign="top">

Decision

</th>
<th valign="top">

Where It Is Documented

</th>
<th valign="top">

Who Owns It

</th>
</tr>
<tr>
<td valign="top">

Supervisor scope: which workflows are in scope (A only / A+B / A+B+C)?

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Domain PM + Discovery Lead

</td>
</tr>
<tr>
<td valign="top">

Multi-agent or single agent? If multi-agent: which topology (1 / 2 / 3)?

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Dev Architect

</td>
</tr>
<tr>
<td valign="top">

Which SAP calculation service backs Layer 2 of the guardrail stack?

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Application Engineer + P&E Expert

</td>
</tr>
<tr>
<td valign="top">

Human approval threshold values for each numerical output

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE team + process owner

</td>
</tr>
<tr>
<td valign="top">

Authorization matrix for all agents

</td>
<td valign="top">

Draft Product Design Specs

</td>
<td valign="top">

FDE Dev Architect + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Regulatory audit trail retention requirement

</td>
<td valign="top">

Legal agreement plan

</td>
<td valign="top">

Customer compliance team (confirmed during Discovery)

</td>
</tr>
<tr>
<td valign="top">

Process KPIs (Dimension 2) for the quality framework

</td>
<td valign="top">

TO-BE process flow

</td>
<td valign="top">

FDE Domain PM + process owner

</td>
</tr>
<tr>
<td valign="top">

Legal agreement path for Workflow C (support relay), if in scope

</td>
<td valign="top">

Legal agreement plan

</td>
<td valign="top">

FDE team + SAP legal

</td>
</tr>
<tr>
<td valign="top">

Cost monitoring budget values and alert owners post-handover

</td>
<td valign="top">

Production handover plan

</td>
<td valign="top">

FDE Domain PM + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Eval test data strategy: synthetic vs. TEA-covered real data

</td>
<td valign="top">

Eval plan

</td>
<td valign="top">

FDE Application Engineer + Data Scientist

</td>
</tr>
</table>

Decisions left unresolved at the end of Discovery become scope risk in the Develop phase. The threshold values and authorization matrices in particular cannot be retrofitted — they shape the workflow design and the Sirius security concept documentation that must be complete before productization.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_sprint_scope"/>

## FDE Sprint Scope Implication

The Starter Kit does not eliminate the FDE sprint's build effort — it removes the infrastructure rebuild that otherwise consumes sprint capacity on work that adds no differentiation for this customer.

**Approximate effort comparison for the supervisor stack (Workflows A, B, C, shared state store)**:

**Table: Effort Comparison With and Without Starter Kit**

<table>
<tr>
<th valign="top">

Activity

</th>
<th valign="top">

Without Starter Kit

</th>
<th valign="top">

With Starter Kit

</th>
</tr>
<tr>
<td valign="top">

Workflow C (Support Relay)

</td>
<td valign="top">

1–2 days build + test

</td>
<td valign="top">

~2 hours: fork, configure, verify sanitization in EAC

</td>
</tr>
<tr>
<td valign="top">

Workflow B (Telemetry Monitor)

</td>
<td valign="top">

2–3 days build + test

</td>
<td valign="top">

~1 day: fork, calibrate thresholds post-EAC baseline, test with injected anomalies

</td>
</tr>
<tr>
<td valign="top">

Workflow A skeleton

</td>
<td valign="top">

2–3 days build

</td>
<td valign="top">

~1 day: fork, author Code node verification rules + critic prompt, test routing

</td>
</tr>
<tr>
<td valign="top">

Shared state store

</td>
<td valign="top">

0.5 days DDL + deploy

</td>
<td valign="top">

~1 hour: deploy from DDL template, verify in each environment

</td>
</tr>
<tr>
<td valign="top">

Audit trail

</td>
<td valign="top">

0.5 days DDL + deploy

</td>
<td valign="top">

~1 hour: deploy from DDL template, verify append-only behavior

</td>
</tr>
</table>

These are estimates for planning purposes, not commitments. The "With Starter Kit" figures for Workflow B, Workflow C, and the shared state store assume the published reference artifact; per the status table above, these three are currently **design targets**, so until they are published the applicable effort is closer to the "Without Starter Kit" column.

The freed capacity goes into the work that actually differentiates the engagement: the worker agent's business logic, the evaluation test suite, integration testing, and the process owner alignment sessions that determine threshold values.

**Build sequence**: the [Joule Studio Build Map](joule-studio-build-map-67ad030.md) build sequence still applies when using the Starter Kit. Fork and configure the supervisor workflows only after the worker agent is stable — not before. A supervisor configured to govern an unstable worker creates debugging noise that slows the sprint.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_agent_hub"/>

## Relationship to SAP AI Agent Hub

The Starter Kit is a build-time and operations-time artifact; the [SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right: is the runtime governance layer. The two are complementary:

-   The Agent Hub's **Agent Registry** records every deployed agent (workers, supervisors, orchestrators) with its identity, version, and authorization scope. The Starter Kit provides the artifacts that get registered.
-   The Agent Hub's **Agent Observability** surfaces the telemetry that Workflow B monitors via the SAP Cloud ALM API. The Starter Kit's Workflow B is the consumer of what the Agent Hub's harness produces.
-   The Agent Hub's **Secure & Govern** capability manages the threshold configuration parameters that the Starter Kit's environment-agnostic configuration pattern exposes as named parameters.
-   The Agent Hub's **Agent Evals & Assessment** runs the eval test suite against the agents the FDE team builds using the Starter Kit worker agent base pattern.

The Starter Kit does not duplicate any of these capabilities. It implements the CIS-side operational layer — the supervisor agent pattern, the audit trail, the escalation logic — that operates within and on top of what the Agent Hub provides.


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_cross_engagement_loop"/>

## Continuous Improvement: From Production Back to the Starter Kit

The continuous improvement loop in the [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md) runs within individual engagements. The Starter Kit adds a second loop that runs across engagements.

```
[Production engagement N]
  CIS T&O identifies improvement in Workflow B rule set,
  audit trail field, or pre-migration checklist item
         ↓
[CIS T&O: assess generalizability]
    ├── Engagement-specific finding → feed into that engagement's improvement loop only
    └── Generalizable finding → propose Starter Kit update
         ↓
[CIS T&O: review and merge into Starter Kit (next version)]
         ↓
[Engagement N+1 forks from updated Starter Kit]
  The improvement is inherited without additional build effort
```

Examples of findings that become Starter Kit improvements:

-   A new BAIP telemetry field becomes available that improves anomaly classification in Workflow B's Code node → add to the reference implementation
-   A production incident reveals that a specific audit trail field was missing and required a schema change mid-engagement → add the field to the base DDL
-   A pre-migration checklist item is consistently missed across engagements → add a more explicit verification step to the checklist
-   A new failure mode is observed across two independent engagements in the same Industry AI domain → add a new rule to the Workflow B rule set


<a name="loioff99fe9cde6447dcb7391ffb3f2fc552__section_summary"/>

## Summary

**Table: Reusability Layer Summary**

<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

What It Contains

</th>
<th valign="top">

Delivery Model

</th>
<th valign="top">

Maintained By

</th>
</tr>
<tr>
<td valign="top">

Layer 1 — Deploy as-is

</td>
<td valign="top">

Workflow C, audit trail schemas, anomaly rule set, sanitization rules, pre-migration checklists, version stack discipline, decision tables, fail-safe rules

</td>
<td valign="top">

Inherited by FDE team; no sprint effort

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Layer 2 — Parameterize and deploy

</td>
<td valign="top">

Workflow B (threshold calibration), Workflow A skeleton (Code node + critic prompt authored per engagement), guardrail stack structure, quality framework, eval structure, rollback template, configuration naming convention

</td>
<td valign="top">

Forked and configured during FDE Develop; parameterization decided in Discovery

</td>
<td valign="top">

CIS T&O (skeleton); FDE team (per-engagement content)

</td>
</tr>
<tr>
<td valign="top">

Layer 3 — Engagement-specific

</td>
<td valign="top">

Worker agent business logic, process KPIs, verification rules, threshold values, authorization matrices, regulatory requirements

</td>
<td valign="top">

Built from scratch in FDE Develop

</td>
<td valign="top">

FDE team (with process owner and customer IT admin)

</td>
</tr>
</table>

The division of labour is clear: CIS T&O invests in making Layers 1 and 2 excellent so that FDE teams can invest their sprint capacity in Layer 3. Layer 3 is where the customer's specific business problem is solved — it is the only layer where CIS cannot substitute a prior answer for the work of understanding this customer.

**Related Information**  


[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

[Proof of Concept Plan](proof-of-concept-plan-d8c5407.md)

[Agent Quality Framework](agent-quality-framework-5f15077.md)

[Precision Guardrails](precision-guardrails-ae75c89.md)

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent registry, access control, lifecycle management, usage telemetry, and policy enforcement.") :arrow_upper_right:

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio — low-code build environment for SAP AI agents, workflows, and tools.") :arrow_upper_right:

[SAP Integration Suite MCP Server](https://help.sap.com/docs/SAP_INTEGRATION_SUITE/9519789d5664487f8b9cd89eba514477/94957bc87247462ebdfd65e72512122a.html "SAP Integration Suite MCP Server — connect SAP Integration Suite capabilities to AI agents via MCP protocol.") :arrow_upper_right:
