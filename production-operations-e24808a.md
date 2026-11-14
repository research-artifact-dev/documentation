<!-- loioe24808a64dad40108f7af27336f7205c -->

# Production Operations

Once an agent is in production and the FDE team has moved on, three operational concerns arise in every long-running Industry AI deployment: agent versioning and rollback, cost and token monitoring, and environment migration. This topic specifies versioning strategy, rollback triggers and mechanics, cost monitoring, the four-environment migration checklist, the AI Golden Path TR1–TR14 mandatory technical requirements, and BTP deployment patterns.


<a name="loioe24808a64dad40108f7af27336f7205c__section_agent_versioning"/>

## Agent Versioning Strategy

### The Version Stack

A deployed agent is not a single artifact — it is a coordinated set of versioned components that must be updated and rolled back as a group.

**The Five-Component Version Stack**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

What It Governs

</th>
<th valign="top">

Where It Is Versioned

</th>
</tr>
<tr>
<td valign="top">

Joule Studio workflow definition

</td>
<td valign="top">

The workflow's node structure, routing logic, and wiring

</td>
<td valign="top">

SAP AI Agent Hub / Joule Studio version history

</td>
</tr>
<tr>
<td valign="top">

SAP Prompt Registry entry

</td>
<td valign="top">

The agent's system prompt (and critic prompts for supervisor)

</td>
<td valign="top">

Git repo + workflow parameter (current); SAP Prompt Registry when `promptTemplate` source ships

</td>
</tr>
<tr>
<td valign="top">

Model selection

</td>
<td valign="top">

Which LLM is used in each `sapOrchestration` node

</td>
<td valign="top">

`sapOrchestration` model parameter (BAIP configuration)

</td>
</tr>
<tr>
<td valign="top">

Threshold configuration

</td>
<td valign="top">

Human approval thresholds, anomaly detection baselines, tolerance bands

</td>
<td valign="top">

n8n workflow variables (`$vars`) in Joule Studio UI — no redeployment required to change

</td>
</tr>
<tr>
<td valign="top">

Authorization scope

</td>
<td valign="top">

Which tools and entities the agent can access

</td>
<td valign="top">

BAIP application registration (policy assignments)

</td>
</tr>
</table>

The most common source of post-deployment quality degradation is updating one component without coordinating the others. Treat a version bump as a version of the entire stack, not of individual components.

### Versioning in Joule Studio

**Current state (node v1.5.0, verified 2026-10-22)**: the `sapOrchestration` node supports only inline prompts (`promptSource: inline`). A `promptTemplate` source — which would enable external registry references — is planned but not yet shipped.

**Prompt versioning fallback approach:**

-   Store prompt text as a **workflow-level string parameter**, configurable per environment via BAIP configuration
-   Maintain the canonical prompt text in the **engagement Git repository** alongside the workflow definition — the commit history is the version history
-   Tag workflow releases with the prompt version in the version stack documentation (e.g., `prompt: v1.2` alongside model and workflow version)
-   Prompt changes require a **workflow redeployment** — treat them as a workflow-level change

### Change Classification

Not all changes carry the same rollback risk. Classify changes before deploying.

**Change Risk Classification**

<table>
<tr>
<th valign="top">

Change Type

</th>
<th valign="top">

Risk Level

</th>
<th valign="top">

Required Before Deploying

</th>
</tr>
<tr>
<td valign="top">

Threshold adjustment

</td>
<td valign="top">

Low

</td>
<td valign="top">

Process owner approval; log in audit trail

</td>
</tr>
<tr>
<td valign="top">

Prompt update (minor wording) — **inline prompt fallback path**

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; workflow redeployment required; treat as Workflow structure change

</td>
</tr>
<tr>
<td valign="top">

Prompt update (structural change to output schema)

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; boundary condition tests; EAC re-validation

</td>
</tr>
<tr>
<td valign="top">

Model version change

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; compare Dimension 1 metrics before/after

</td>
</tr>
<tr>
<td valign="top">

Workflow structure change

</td>
<td valign="top">

High

</td>
<td valign="top">

Full eval run; security concept update; process owner review

</td>
</tr>
<tr>
<td valign="top">

Authorization scope change

</td>
<td valign="top">

Very high

</td>
<td valign="top">

DPCE (Data Protection and Compliance Engineering) review; CA assessment if scope expands

</td>
</tr>
</table>

**Inline prompt risk collapse**: as of mid-2026, `promptTemplate` is not yet available in the `sapOrchestration` node. Agents built before it becomes available must embed prompts inline in the workflow, collapsing even minor wording changes to the same risk tier as Workflow structure changes because any prompt edit requires a workflow redeployment. When `promptTemplate` becomes available, migrate inline prompts to the Registry as a planned sprint item and reclassify prompt updates to the Medium risk tier.

### Multi-Agent Version Coordination

In a multi-agent system, each worker agent has its own version stack. Rolling back one worker agent may break its interface contract with the orchestrator or another worker agent if they exchange structured outputs.

Before executing any version change or rollback in a multi-agent deployment:

-   **Map interface contracts**: any structured output from the affected agent consumed by another agent is an interface contract. Document these contracts in the release plan.
-   **Check schema compatibility**: confirm that the changed version's output schema is compatible with every consuming agent's current version.
-   **Roll back as a group when in doubt**: treat the multi-agent system as a coordinated stack and roll back all affected agents together.

The CIS T&O operating runbook should include a dependency map of inter-agent interface contracts for any multi-agent deployment.


<a name="loioe24808a64dad40108f7af27336f7205c__section_rollback"/>

## Rollback: Triggers and Mechanics

### Rollback Triggers

A rollback is warranted when production quality metrics show a confirmed regression correlated with a version change.

**Rollback Trigger Signals**

<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
<th valign="top">

Source

</th>
</tr>
<tr>
<td valign="top">

Human override rate increase

</td>
<td valign="top">

> 20 percentage points above pre-deployment baseline

</td>
<td valign="top">

Audit trail `delta` field

</td>
</tr>
<tr>
<td valign="top">

Verification rejection rate increase

</td>
<td valign="top">

> 15 percentage points above pre-deployment baseline

</td>
<td valign="top">

Supervisor Workflow A telemetry

</td>
</tr>
<tr>
<td valign="top">

Process KPI degradation

</td>
<td valign="top">

> 20% below pre-deployment baseline

</td>
<td valign="top">

Process owner report or Agent Hub process KPI tracking

</td>
</tr>
<tr>
<td valign="top">

Structured output format failures

</td>
<td valign="top">

> 5% of invocations producing malformed output

</td>
<td valign="top">

BAIP harness error rate telemetry

</td>
</tr>
<tr>
<td valign="top">

LLM refusal rate

</td>
<td valign="top">

Any non-zero refusal rate in production (zero expected baseline)

</td>
<td valign="top">

SAP Cloud ALM error classification

</td>
</tr>
</table>

**Correlation requirement**: before triggering a rollback, confirm that the signal change is correlated with a version deployment event. The audit trail's `agent_version` field makes this correlation explicit — every committed output carries the agent version that produced it.

### Rollback Mechanics

**Prompt rollback (current state — inline prompts):** A prompt change or rollback requires a **workflow redeployment** today — prompts are stored inline as a workflow-level parameter. Treat it as a workflow-level change and apply the workflow-rollback mechanics below. In either case, audit the prompt diff to understand what change caused the regression.

**Model rollback:**

-   Change the `model` parameter in the `sapOrchestration` node's BAIP configuration
-   No workflow redeployment required (if model is referenced by name as BAIP configuration, not hardcoded in the workflow node)
-   Verify that the prior model version is still available in the generative AI hub (model deprecation can prevent rollback — check SAP Note 3437766 for model availability)

**Workflow rollback:**

-   Restore the prior workflow version from Joule Studio / Agent Hub version history
-   Requires a deployment operation; more disruptive than model rollback
-   Confirm that the prior workflow version is compatible with the current authorization scope

**Threshold rollback:**

-   Restore prior threshold values in the n8n Variables UI (`$vars`)
-   Immediate effect; no deployment required
-   The change must be logged in the audit trail as a configuration change event

**Full stack rollback** (last resort): restore all five components to the prior coordinated version. Execute in the order: threshold → model → prompt → authorization scope → workflow (workflow last, as it is the most disruptive).

### State Continuity During Rollback

Rollback does not erase in-flight transactions or the audit trail:

-   **In-flight transactions**: before relying on graceful drain, **confirm it is supported by the BAIP managed runtime configuration for the customer's tenant**. If graceful drain is **not** supported, execute an explicit hard cutover: (1) pause new invocations; (2) wait for in-flight sessions to complete (monitor active session count; allow up to 5 minutes); (3) execute the rollback; (4) re-enable invocations.
-   **Audit trail**: append-only and version-tagged. Do not delete or modify audit trail records during rollback — they are the evidence needed to understand what went wrong.
-   **Shared state store**: the supervisor's rolling baseline should be reset after a rollback. The baseline built up under the degraded version is not representative of the rolled-back version's behavior.
-   **Human approval queue**: any pending `sapTaskCenter` tasks created by the degraded version should be reviewed by the process owner before approval.

### Rollback Authorization

**Rollback Authorization by Topology**

<table>
<tr>
<th valign="top">

Topology

</th>
<th valign="top">

Rollback Authorization

</th>
</tr>
<tr>
<td valign="top">

Customer tenant (Topology A)

</td>
<td valign="top">

Customer IT admin authorizes; CIS T&O proposes and may execute with consent

</td>
</tr>
<tr>
<td valign="top">

SAP managed tenant (Topology B)

</td>
<td valign="top">

CIS T&O can execute rollback directly as the operator; notify customer

</td>
</tr>
</table>

For all rollbacks: document the trigger, the scope of the rollback, the authorization, and the post-rollback verification result in the CIS T&O operating log.

### Model Lifecycle and Deprecation

Model deprecation is an architectural risk, not just a rollback edge case. SAP Note 3437766 is the authoritative source for model availability, deprecation notices, and retirement dates.

**Two version modes** — choose deliberately at build time:

-   `latest`: automatically moved to the newest model version when the current one is deprecated. Model behavior can change without a deployment event, making regressions harder to correlate and rollback impossible.
-   **Pinned version** (recommended for production): the specific model version is used until explicitly changed.

**Proactive model lifecycle strategy:**

1.   **Monitor** SAP Note 3437766 actively. When a successor model appears or a deprecation date is set, treat it as a sprint-level work item.
2.   **Test the successor** against the engagement's eval suite before the current version is deprecated. Pay particular attention to structured output formatting, refusal behavior, and precision-sensitive outputs.
3.   **Version the model reference** in the `sapOrchestration` BAIP configuration as part of the five-component version stack.
4.   **Document the validated successor** in the CIS T&O operating log for the engagement.

### Emergency Procedure When Model Rollback Is Impossible

If the current pinned model is deprecated with no advance notice and the validated successor is not yet in place, rollback is impossible. SAP Note 3437766 explicitly states that some model retirements occur without an advance deprecation period.

**Emergency response:**

1.   **Activate conservative LLM critic configuration**: tighten the LLM critic's tolerance to the most conservative setting (typically ±5% of the reference value for numerical outputs). This increases false-positive escalations to `sapTaskCenter` but prevents incorrect outputs from committing. This is a `$vars` configuration change (Low risk tier) — it does not require workflow redeployment.
2.   **Run a compressed eval against the successor**: prioritize the boundary condition test cases — these are the scenarios most likely to expose behavioral differences.
3.   **Authorization**: model configuration changes under this procedure require authorization from the customer IT admin (Topology A) or CIS T&O operator (Topology B).
4.   **Customer communication**: notify the customer's Business Lead and IT Admin that an unexpected model change has occurred and that conservative guardrails are active.
5.   **Restore normal operation**: once the compressed eval confirms that the successor model meets the pre-deprecation success rate baseline, restore the LLM critic to the calibrated tolerance and update the version stack.


<a name="loioe24808a64dad40108f7af27336f7205c__section_cost_token_monitoring"/>

## Cost and Token Monitoring

### CALM Monitoring Baseline at Customer Sites

> ### Note:  
> CALM-based monitoring cannot be assumed to be available in customer environments and must not be used as the default production monitoring baseline for CAE (Cloud Application and Extension) deployments. PSR update CISATQ-131 (2026-09-09) formally documents this constraint.

The production monitoring baseline for Industry AI / CAE agents must be agreed as a project-specific requirement with each customer during FDE Discovery. The default is a combination of the agent's built-in OTel telemetry, whatever monitoring infrastructure the customer already has available, and cost and quality signals surfaced by Supervisor Workflow B.

### Synthetic Monitoring

SAP operates three tools for platform-level synthetic monitoring:

-   **BTP Availability Service**: active endpoint probing for BTP-deployed services
-   **Site 24/7**: cross-region uptime monitoring for agent-accessible endpoints
-   **Monterey**: SAP internal synthetic monitoring platform for core infrastructure components

The FDE team's synthetic monitoring responsibility is limited to: confirming the agent's A2A endpoint is reachable (step 7 of the 9-step deploy loop) and including a reachability verification step in the EAC → Production migration checklist.

### Agentic Remediation (Topology B Only)

For agents deployed in **Topology B** (SAP-managed tenant), SAP SRE operates **Agentic Remediation**: a proactive capability that detects platform-level issues affecting agent availability and takes automated remediation steps. This reduces the operational burden on CIS T&O for infrastructure-level incidents. CIS T&O's scope in Topology B remains application-layer quality monitoring.

### Two Cost Pools: Build-Time vs. Runtime

Token costs in Industry AI delivery split into two separate pools. **Build-time costs** are incurred by FDE engineers using Claude Code and AI coding tools during the sprint; these are tracked via the SAP-internal Hyperspace LLM Proxy. **Runtime costs** are incurred by deployed agents processing business transactions in the customer's BAIP tenant; these are measured via AI Units and OpenTelemetry token counts. This section covers runtime cost monitoring exclusively. Build-time efficiency guidelines are in [Token Efficiency](token-efficiency-8449831.md).

### What to Monitor

Token consumption is both a billing concern and a quality and operational signal: runaway loops consume tokens exponentially before quality signals fire; context expansion creep drifts upward as production use grows; model changes alter output verbosity.

**Token Monitoring Metrics and Thresholds**

<table>
<tr>
<th valign="top">

Metric

</th>
<th valign="top">

What It Measures

</th>
<th valign="top">

Alert Threshold

</th>
</tr>
<tr>
<td valign="top">

Input tokens per invocation

</td>
<td valign="top">

Context size; grounding data volume

</td>
<td valign="top">

> 2× p95 baseline for 5 consecutive invocations

</td>
</tr>
<tr>
<td valign="top">

Output tokens per invocation

</td>
<td valign="top">

Response verbosity; structured output completeness

</td>
<td valign="top">

> 3× p95 baseline for 3 consecutive invocations

</td>
</tr>
<tr>
<td valign="top">

Total tokens per invocation

</td>
<td valign="top">

Composite cost signal

</td>
<td valign="top">

> 3× p95 baseline (runaway indicator)

</td>
</tr>
<tr>
<td valign="top">

Cost per invocation

</td>
<td valign="top">

Token count × model price rate

</td>
<td valign="top">

> 3× p95 baseline

</td>
</tr>
<tr>
<td valign="top">

Cost per committed output

</td>
<td valign="top">

Total cost ÷ outputs successfully written to system of record

</td>
<td valign="top">

Trend upward > 30% over 30-day window

</td>
</tr>
<tr>
<td valign="top">

Daily total cost

</td>
<td valign="top">

Budget tracking

</td>
<td valign="top">

> 150% of expected daily budget (alert IT admin)

</td>
</tr>
</table>

### AI Units: The Commercial Billing Layer

The commercial billing model for deployed FDE agents uses **AI Units per agent step**, charged through the SAP AI Units consumption model.

**AI Unit Tiers**

<table>
<tr>
<th valign="top">

Step Type

</th>
<th valign="top">

AI Units / Step

</th>
<th valign="top">

Net Price Range (EUR)

</th>
</tr>
<tr>
<td valign="top">

Basic (simple lookups, single tool calls, content retrieval)

</td>
<td valign="top">

0.005

</td>
<td valign="top">

€0.013–€0.035

</td>
</tr>
<tr>
<td valign="top">

Standard (multi-tool orchestration, moderate reasoning)

</td>
<td valign="top">

0.010

</td>
<td valign="top">

€0.026–€0.070

</td>
</tr>
<tr>
<td valign="top">

Advanced (complex reasoning, multi-system orchestration, voice)

</td>
<td valign="top">

0.025

</td>
<td valign="top">

€0.066–€0.175

</td>
</tr>
</table>

**"Agent step" and "Agent Action" are the same metering unit.** The floor-pricing model calls this unit an **Agent Action**, defined as 1 LLM call + 1 simple tool invocation = 1 Agent Action.

**Floor pricing gate timing**: floor pricing applies at commercial release, not at sprint start. Initiate the formal submission at week 8, when the step count and step tier are stable.


<a name="loioe24808a64dad40108f7af27336f7205c__section_cost_monitoring_implementation"/>

## Cost Monitoring Implementation

### Extending Supervisor Workflow B

The most practical implementation extends Supervisor Workflow B to include token consumption alongside quality signals.

```
[Workflow B: Schedule Trigger]
         ↓
[HTTP Request: SAP Cloud ALM API — retrieve traces from last polling window]
[HTTP Request: SAP AI Core usage API — retrieve token counts for same window]
         ↓
[Code node: join traces with token counts by correlation_id]
         ↓
[Code node: existing quality anomaly checks]
[Code node: token anomaly checks]
  - per_invocation_tokens vs. rolling_baseline → flag if > 3× for 3 consecutive
  - runaway_detection: single invocation > 5× p99 → immediate alert
  - daily_total_cost vs. daily_budget → flag if > 150%
  - cost_per_committed_output trend → flag if 30-day trend > 30% upward
         ↓
[Decision routing: same as existing Workflow B]
    ├── No anomaly → update baseline
    ├── Cost anomaly → sapTaskCenter alert to IT admin (with cost details)
    └── Runaway indicator → immediate sapTaskCenter (VERY_HIGH priority) + trigger Workflow C
```

**Runaway loop handling**: a single-invocation token spike (> 5× p99) is treated as a Critical anomaly. The agent should be suspended via the Agent Hub Secure & Govern capability pending investigation.

**ServiceNow as alert target**: for customers whose primary IT service management platform is ServiceNow, Workflow B alerts can be routed to ServiceNow incidents via SAP Cloud Service Management. Confirm the customer's ITSM platform and preferred routing in FDE Discovery — include the alert target decision in the Draft Product Design Specs.

### Token Budget Hard Cap via sapOrchestration

Set `max_tokens` in the `sapOrchestration` node to 2× the p95 output token count observed during EAC. An agent that hits the token ceiling mid-response will produce a malformed structured output, which the Layer 2 deterministic check will catch and escalate to `sapTaskCenter` — the correct fail-safe behavior.

### Audit Trail Cost Extension

The audit trail should include token consumption fields for committed outputs.

**Audit Trail Cost Fields**

<table>
<tr>
<th valign="top">

Field

</th>
<th valign="top">

Content

</th>
</tr>
<tr>
<td valign="top">

`input_tokens`

</td>
<td valign="top">

Input token count for the primary `sapOrchestration` call in this invocation

</td>
</tr>
<tr>
<td valign="top">

`output_tokens`

</td>
<td valign="top">

Output token count for the primary `sapOrchestration` call

</td>
</tr>
<tr>
<td valign="top">

`total_tokens_invocation`

</td>
<td valign="top">

Sum of all `sapOrchestration` calls in this invocation (worker + critic)

</td>
</tr>
<tr>
<td valign="top">

`model_used`

</td>
<td valign="top">

Model identifier used in this invocation

</td>
</tr>
</table>

**ALS v3 NG — platform-level immutable audit log**: the business-focused audit trail captures decision-quality data for CIS T&O use. **ALS v3 NG** (Audit Log Service Next Generation) provides the platform-level immutable audit log for agent identity events, security-relevant actions, and authorization decisions — the compliance-grade source for regulatory audits. Confirm which is required for the customer's compliance framework during FDE Discovery.

### Standard Observability Middleware Integration

The recommended observability pattern for BTP-deployed agents is a drop-in Python OTel middleware package (available in Project LightSpeed at `references/middleware/`). It requires two lines of setup code:

```python
from middleware import setup_observability
setup_observability("my-agent-service")
```

Once initialized, it emits structured JSON logs on stdout and OTel traces via OTLP. The middleware resolves its export destination using the following order of preference:

1.   `OTEL_EXPORTER_OTLP_ENDPOINT` environment variable — used for custom OTLP endpoints (sovereign region deployments)
2.   Cloud Foundry `cloud-logging` service binding — used in standard SAP regions with SAP Cloud Logging
3.   Disabled / logging-only fallback — used in CN40, canary, and any environment where neither of the above is configured; the middleware starts without error and emits to stdout only

This resolution order means the same middleware code works unchanged across local development, standard region deployment, and sovereign region deployment.


<a name="loioe24808a64dad40108f7af27336f7205c__section_environment_migration"/>

## Environment Migration

### The Four Environments

**FDE Engagement Environment Sequence**

<table>
<tr>
<th valign="top">

Environment

</th>
<th valign="top">

Purpose

</th>
<th valign="top">

Who Operates

</th>
</tr>
<tr>
<td valign="top">

Developer sandbox

</td>
<td valign="top">

Initial build, rapid iteration, no governance constraints

</td>
<td valign="top">

FDE Application Engineer

</td>
</tr>
<tr>
<td valign="top">

Internal test

</td>
<td valign="top">

Integration testing, eval runs, security review

</td>
<td valign="top">

FDE team (all roles)

</td>
</tr>
<tr>
<td valign="top">

Customer non-production (EAC)

</td>
<td valign="top">

Customer validation with Early Adopter Care contacts; behavioral baseline establishment

</td>
<td valign="top">

FDE team + customer IT admin

</td>
</tr>
<tr>
<td valign="top">

Customer production

</td>
<td valign="top">

Live operation; handover complete

</td>
<td valign="top">

Customer IT admin + CIS T&O

</td>
</tr>
</table>

### What Moves Between Environments — and What Does Not

**Artifact Migration Rules**

<table>
<tr>
<th valign="top">

Artifact

</th>
<th valign="top">

Moves?

</th>
<th valign="top">

How

</th>
<th valign="top">

Notes

</th>
</tr>
<tr>
<td valign="top">

Joule Studio workflow definition

</td>
<td valign="top">

Yes — export

</td>
<td valign="top">

Workflow export from source env; import in target env

</td>
<td valign="top">

Credentials are NOT included in exports — re-provision per environment

</td>
</tr>
<tr>
<td valign="top">

Eval test suite

</td>
<td valign="top">

Yes — same suite in all envs

</td>
<td valign="top">

Git-versioned alongside the concept; runs unchanged in each environment

</td>
<td valign="top">

Synthetic test data must be available in each environment

</td>
</tr>
<tr>
<td valign="top">

BAIP application registrations (agent identities)

</td>
<td valign="top">

No — re-create

</td>
<td valign="top">

New registration per environment

</td>
<td valign="top">

Each environment needs a separate identity with appropriate scoped authorizations

</td>
</tr>
<tr>
<td valign="top">

Authorization scope definitions

</td>
<td valign="top">

No — re-define

</td>
<td valign="top">

Tighter scope in production than in dev/test; prod scope reviewed and approved

</td>
<td valign="top">

Dev may have broader read access for debugging; prod should be minimum necessary

</td>
</tr>
<tr>
<td valign="top">

Threshold configuration parameters

</td>
<td valign="top">

No — re-set

</td>
<td valign="top">

Copy dev values as starting point; calibrate prod values from EAC behavioral data

</td>
<td valign="top">

Never copy dev thresholds directly to production

</td>
</tr>
<tr>
<td valign="top">

HANA Cloud connection credentials

</td>
<td valign="top">

No — re-provision

</td>
<td valign="top">

New service credentials per environment

</td>
<td valign="top">

Never copy credentials between environments

</td>
</tr>
<tr>
<td valign="top">

Shared state store schema

</td>
<td valign="top">

No — re-create

</td>
<td valign="top">

DDL script creates the schema; initial state is empty in each environment

</td>
<td valign="top">

State does NOT migrate — each environment builds its own baseline from scratch

</td>
</tr>
<tr>
<td valign="top">

Supervisor Workflow B baseline

</td>
<td valign="top">

No — rebuild

</td>
<td valign="top">

Allow 48–72 hours of observation in new environment before enabling anomaly alerts

</td>
<td valign="top">

Imported baseline from prior environment is invalid in new environment

</td>
</tr>
</table>

**The three things that must never be copied between environments**: credentials and API keys; dev/test thresholds directly to production; agent identities (BAIP application registrations).

### Environment-Specific Configuration Pattern

The n8n workflow definition should be environment-agnostic. All environment-specific values should be injected at runtime through:

-   **Named BAIP credential objects** for system connections, API keys, service bindings
-   **n8n workflow variables (`$vars`)** for static, per-environment values such as thresholds, model selection, and recipient IDs — set in the n8n Variables UI without redeployment
-   **Trigger-payload config (`$json.config.x`)** for per-invocation values

> ### Note:  
> `{{config.x}}` is **not a valid n8n expression** — it resolves to `undefined` at runtime. The correct syntax is `{{$vars.x}}` for instance-level variables or `{{$json.config.x}}` for trigger-payload config.

```
sapOrchestration configuration (environment-agnostic):
  model:          {{$vars.modelSelection}}              ← n8n variable
  systemPrompt:   {{$vars.systemPrompt}}                ← n8n variable
  connection:     {{credentials.sap_system_connection}} ← from BAIP credential object

sapTaskCenter configuration:
  recipients:     {{$vars.approvalRecipients}}          ← n8n variable

Threshold parameters:
  approval_threshold:  {{$vars.approvalThreshold}}      ← n8n variable (calibrated per env)
  tolerance_band:      {{$vars.toleranceBand}}          ← n8n variable
```

**Environment-Specific Values That Change Per Environment**

<table>
<tr>
<th valign="top">

Parameter

</th>
<th valign="top">

Dev

</th>
<th valign="top">

Internal Test

</th>
<th valign="top">

EAC (Non-Prod)

</th>
<th valign="top">

Production

</th>
</tr>
<tr>
<td valign="top">

SAP system endpoint

</td>
<td valign="top">

Dev S/4HANA

</td>
<td valign="top">

Test S/4HANA

</td>
<td valign="top">

Customer non-prod S/4HANA

</td>
<td valign="top">

Customer prod S/4HANA

</td>
</tr>
<tr>
<td valign="top">

Approval threshold

</td>
<td valign="top">

Permissive (for testing)

</td>
<td valign="top">

Mid-range (for eval coverage)

</td>
<td valign="top">

Calibrated from initial observation

</td>
<td valign="top">

Calibrated from EAC baseline

</td>
</tr>
<tr>
<td valign="top">

Task Center recipient

</td>
<td valign="top">

FDE engineer userId

</td>
<td valign="top">

FDE team group

</td>
<td valign="top">

Customer process owner (UAT)

</td>
<td valign="top">

Customer process owner (prod)

</td>
</tr>
<tr>
<td valign="top">

Anomaly detection thresholds

</td>
<td valign="top">

Not active in dev sandbox

</td>
<td valign="top">

Conservative (broad detection for testing)

</td>
<td valign="top">

Calibrated after 48h observation

</td>
<td valign="top">

Calibrated from EAC baseline

</td>
</tr>
</table>


<a name="loioe24808a64dad40108f7af27336f7205c__section_pre_migration_checklist"/>

## Pre-Migration Checklist

Before each environment transition, verify the following.

**Dev → Internal Test:**

-   All eval test cases pass in dev with documented acceptance thresholds
-   Credential handling reviewed: no hardcoded secrets in any workflow node
-   BAIP application registration created for internal test environment
-   Authorization scope reviewed and tightened from dev (remove any debug-scope permissions)
-   Shared state store schema deployed in test environment (empty initial state)
-   Audit trail table schema deployed in test environment

**Internal Test → EAC (Customer Non-Production):**

-   Full eval suite passes with production-equivalent data
-   Security concept documentation complete and DPCE review initiated
-   New BAIP application registrations for EAC environment (separate from internal test)
-   Customer IT admin has received and confirmed role assignments in Joule Studio
-   Legal agreement confirmed: CA required if EAC uses personal or productive customer data
-   Supervisor Workflow B deployed and tested in EAC environment
-   Task Center recipients configured with actual customer process owner user IDs
-   Runaway detection enabled (token spike alert via Workflow B)

**EAC → Production:**

-   EAC behavioral baseline captured (48–72h minimum observation period complete)
-   **Go-live semantic quality baseline captured** (if drift monitoring is in scope): multi-dimensional quality snapshot taken across the final two weeks of EAC — task completion rate, escalation rate, LLM-judge faithfulness scores, and output embedding centroid, stored as a signed per-customer-agent artifact in SAP AI Core. Requires minimum 500 production interactions for statistical validity; confirm expected interaction volume in Discovery before committing to drift monitoring.
-   Production thresholds calibrated from EAC baseline and agreed with process owner
-   Sirius productization tasks complete: DPCE threat model, security concept, QPA test strategy, AI ethics assessment
-   Production BAIP application registrations created with minimum-necessary scopes
-   Audit trail retention policy confirmed with customer compliance team
-   Supervisor Workflow B anomaly baselines reset (will rebuild from production observations)
-   If Workflow B routes to Cloud ALM: Cloud ALM LMS service registration complete (`serviceId` obtained) and `CALM_SERVICE_ID` configured as env var in BTP Cockpit for runtime injection
-   CIS T&O operational runbook delivered as part of handover package (see [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md))
-   Production handover documentation complete (agent authorization matrix, threshold configuration ownership, escalation paths)


<a name="loioe24808a64dad40108f7af27336f7205c__section_fde_delivery_implications"/>

## FDE Delivery Implications

### During Discovery

-   Confirm how many environments exist in the customer's landscape and whether a non-production tenant for EAC is available.
-   Identify the customer IT admin who will manage BAIP application registrations and threshold configuration in production. This person should be engaged in Discovery, not introduced at handover.
-   If the engagement will route Workflow B telemetry to Cloud ALM: confirm the customer has Cloud ALM provisioned and that the IT admin can complete the 5-step LMS service registration (register service → obtain `serviceId` → configure destination → assign agent → validate ingestion). The `serviceId` is tenant-specific and cannot be predetermined — confirm in Discovery to avoid a late-sprint blocker at EAC go-live.
-   Decide in Discovery whether **post-deploy semantic drift monitoring** is in scope. If yes: OTel instrumentation including `gen_ai.conversation.id` trace correlation (TR12) must be active from sprint week 1; plan the EAC stabilization window to reach the minimum 500 production interactions required for a statistically valid go-live baseline.

### During Develop

-   The FDE sprint has a hard cap of **300 Person Days (PD) and 90 calendar days**. Environment migration tasks must be planned within this budget.
-   Build the configuration parameter framework from Week 0, not as a post-MVP activity.
-   Run the eval suite in each environment, not just in dev.
-   Start **AI Onboarding (AI101 task)** early in Development — the AI Feature ID is required before EAC launch and the onboarding process takes several weeks.

### During Productization

**Productization starts at RTC EAC (first go-live).** The productization track must be prepared and the formal submission initiated during the Develop phase — not deferred to after EAC completion.

-   Productization takes **15 days to 3 months** from RTC EAC to GA, depending on the GA path: Microdelivery (LoB leads) or Independent Release (Application AI leads end-to-end).
-   Document the rollback procedure explicitly in the T&O operational runbook: what triggers a rollback, who authorizes it, the step-by-step mechanics, and the post-rollback verification steps.
-   **Adoption Strategy** is a required handover artifact alongside the technical runbook. Before productization handover is signed off, the Adoption Strategy must be complete: all 7 sections drafted, all 9 governance roles named, and the Executive Sponsor signature obtained.
-   **Enable track exit criteria** must be met before handover is complete. At minimum, the customer IT admin must have reached Stage 3 (Reverse Shadowing).


<a name="loioe24808a64dad40108f7af27336f7205c__section_tr1_tr14"/>

## Production Shipping Readiness — TR1–TR14

Every agent developed in an FDE sprint that will be productized as a commercial SAP offering must satisfy the AI Golden Path's 14 mandatory Technical Requirements before commercial release. The FDE Development phase should build toward TR1–TR14 compliance from sprint week 1; retroactive compliance is expensive and often requires architectural rework.

### The Checklist

**TR1–TR14 Technical Requirements**

<table>
<tr>
<th valign="top">

TR

</th>
<th valign="top">

Requirement

</th>
<th valign="top">

FDE Implication

</th>
</tr>
<tr>
<td valign="top">

TR1

</td>
<td valign="top">

A2A Server exposure

</td>
<td valign="top">

Agent must expose an A2A protocol endpoint. Required for eval harness integration and ecosystem interoperability

</td>
</tr>
<tr>
<td valign="top">

TR2

</td>
<td valign="top">

AppFND (Application Foundation) runtime

</td>
<td valign="top">

Agents run on BTP Kyma via AppFND. A formal exception process exists but requires justification

</td>
</tr>
<tr>
<td valign="top">

TR3

</td>
<td valign="top">

IAS app2app tokens

</td>
<td valign="top">

No API keys or username/password for agent-to-agent calls. IAS app2app tokens throughout

</td>
</tr>
<tr>
<td valign="top">

TR4

</td>
<td valign="top">

Unified Services onboarding

</td>
<td valign="top">

SAP-managed provisioning via Unified Metadata Service (UMS). Required for Agent Gateway routing

</td>
</tr>
<tr>
<td valign="top">

TR5

</td>
<td valign="top">

UCL onboarding

</td>
<td valign="top">

SAP-managed integrations. Required for production deployment

</td>
</tr>
<tr>
<td valign="top">

TR6

</td>
<td valign="top">

ORD with `sap:ai:v1` policy level

</td>
<td valign="top">

ORD endpoint published with AI policy level; agent namespace must use `sap.agt*` pattern. Without this, the agent is invisible to Agent Gateway

</td>
</tr>
<tr>
<td valign="top">

TR7

</td>
<td valign="top">

SPII for Agent Gateway

</td>
<td valign="top">

Bi-directional IAS app2app trust for Agent Gateway. Required for tool invocation routing

</td>
</tr>
<tr>
<td valign="top">

TR8

</td>
<td valign="top">

BTP Test Blueprint

</td>
<td valign="top">

Test blueprint covering functional, security, and performance validation

</td>
</tr>
<tr>
<td valign="top">

TR9

</td>
<td valign="top">

Offline + online evaluations (aeval)

</td>
<td valign="top">

Offline eval suite in CI before each release; online eval monitoring active in production. See [Agent Evaluation](agent-evaluation-bb08bc0.md)

</td>
</tr>
<tr>
<td valign="top">

TR10

</td>
<td valign="top">

Agent Gateway for all tool/agent calls

</td>
<td valign="top">

Every tool invocation and agent-to-agent call routes through Agent Gateway — no direct external API calls bypass governance

</td>
</tr>
<tr>
<td valign="top">

TR11

</td>
<td valign="top">

MCP servers for tool calls

</td>
<td valign="top">

All tool calls use MCP protocol. `sapMcpClient` in Joule Studio satisfies this for SAP system connections

</td>
</tr>
<tr>
<td valign="top">

TR12

</td>
<td valign="top">

OpenTelemetry with GenAI semantic conventions

</td>
<td valign="top">

Mandatory, not recommended. Enables token monitoring, evaluation metrics, and the audit trail

</td>
</tr>
<tr>
<td valign="top">

TR13

</td>
<td valign="top">

Metering via OctoRoute + Unified Metering Service

</td>
<td valign="top">

Revenue-critical. Origin propagated via `x-sap-origin` header; `sap.ai.agent.trigger.type` in metering payload. Origin propagates transitively through agent-to-agent calls

</td>
</tr>
<tr>
<td valign="top">

TR14

</td>
<td valign="top">

Extensibility support

</td>
<td valign="top">

Every production agent must support Prompt/Tool Extensions, Pre-Extension Hooks, and Post-Extension Hooks with configurable timeout and circuit-breaker failure policies

</td>
</tr>
</table>

### Sprint Integration Points

TR1–TR14 compliance is a design constraint from week 1, not a productization-phase check.

**TR Readiness by Sprint Phase**

<table>
<tr>
<th valign="top">

Sprint Phase

</th>
<th valign="top">

TR Readiness Targets

</th>
</tr>
<tr>
<td valign="top">

Week 0 — Architecture

</td>
<td valign="top">

A2A endpoint design (TR1); AppFND runtime confirmed (TR2); ORD namespace reserved (TR6)

</td>
</tr>
<tr>
<td valign="top">

Weeks 1–6 — MVP

</td>
<td valign="top">

IAS app2app tokens configured (TR3); OTel instrumentation active (TR12); Agent Gateway routing established (TR10, TR11)

</td>
</tr>
<tr>
<td valign="top">

Weeks 7–9 — Pre-deployment

</td>
<td valign="top">

UMS/UCL onboarding started (TR4, TR5); offline eval in CI (TR9); metering payload implemented (TR13); extensibility hooks defined (TR14)

</td>
</tr>
<tr>
<td valign="top">

Weeks 9–12 — EAC

</td>
<td valign="top">

SPII configured (TR7); BTP Test Blueprint complete (TR8); online eval active (TR9); TR self-assessment signed off

</td>
</tr>
</table>

**Floor pricing approval** is mandatory before every commercial release. Initiate no later than week 8 of the sprint. Agent Actions (1 LLM call + 1 simple tool invocation = 1 Agent Action) are the metering unit.

### AI Agent Hub Production Gate

**AI Agent Hub approval is a mandatory production gate** before the agent enters customer production. This gate is distinct from TR1–TR14 compliance: TR compliance addresses technical readiness; AI Agent Hub approval addresses governance registration, agent identity provisioning, and lifecycle enrollment.

Initiate AI Agent Hub registration no later than week 9:

-   Register agent identity in the **Agent and MCP Registry**
-   Assign policy bindings and scoped authorization grants
-   Confirm tenant isolation, observability routing, and Agent Evals and Assessment enrollment

### Group 1 / Group 2 Deprecation Classification

SAP's formal AI use case lifecycle policy classifies deprecated AI use cases into two groups.

**Deprecation Classification**

<table>
<tr>
<th valign="top">

Classification

</th>
<th valign="top">

Policy

</th>
<th valign="top">

Implication for FDE-Delivered Agents

</th>
</tr>
<tr>
<td valign="top">

**Group 1**

</td>
<td valign="top">

Customer continues using the deprecated capability through the end of their current cloud subscription term; no forced migration during the contract period

</td>
<td valign="top">

CIS commits to supporting the agent through the subscription term; microdeliveries continue as planned

</td>
</tr>
<tr>
<td valign="top">

**Group 2**

</td>
<td valign="top">

6-month transition period from deprecation announcement; migration to the replacement capability is mandatory after 6 months

</td>
<td valign="top">

CIS must plan and support a migration path; the CoE operating rhythm must include migration sprint planning within the 6-month window

</td>
</tr>
</table>


<a name="loioe24808a64dad40108f7af27336f7205c__section_btp_deployment_patterns"/>

## BTP Deployment Patterns

### Nine-Step Autonomous Deploy Loop

The Project LightSpeed deploy pattern (`/fde-deploy`) provides a repeatable 9-step procedure for BTP Kyma agent deployments. FAIL-level guardrail failures abort the loop at step 3.

**Nine-Step Deploy Loop**

<table>
<tr>
<th valign="top">

Step

</th>
<th valign="top">

Action

</th>
<th valign="top">

Abort Condition

</th>
</tr>
<tr>
<td valign="top">

1

</td>
<td valign="top">

Read deployment profile (`pilot.yaml`)

</td>
<td valign="top">

Profile missing or malformed

</td>
</tr>
<tr>
<td valign="top">

2

</td>
<td valign="top">

Verify BTP environment — subaccount, entitlements, service instances

</td>
<td valign="top">

Required service instances not present

</td>
</tr>
<tr>
<td valign="top">

3

</td>
<td valign="top">

Run 21-check deploy guardrails (FAIL = abort, WARN = continue with log)

</td>
<td valign="top">

Any FAIL-level check fails

</td>
</tr>
<tr>
<td valign="top">

4

</td>
<td valign="top">

Build + push container image (3 retries on transient failure)

</td>
<td valign="top">

Build or push fails after 3 retries

</td>
</tr>
<tr>
<td valign="top">

5

</td>
<td valign="top">

Apply Kubernetes manifests (or CF push for Cloud Foundry deployments)

</td>
<td valign="top">

Apply fails; rollback initiated

</td>
</tr>
<tr>
<td valign="top">

6

</td>
<td valign="top">

Backend service deployment

</td>
<td valign="top">

Service deployment fails

</td>
</tr>
<tr>
<td valign="top">

7

</td>
<td valign="top">

A2A smoke test — verify agent responds on the A2A endpoint

</td>
<td valign="top">

A2A endpoint unreachable or returns error

</td>
</tr>
<tr>
<td valign="top">

8

</td>
<td valign="top">

Joule capability registration — connect agent to Joule

</td>
<td valign="top">

Registration fails or capability bundle rejected

</td>
</tr>
<tr>
<td valign="top">

9

</td>
<td valign="top">

Update deployment profile and generate status report

</td>
<td valign="top">

Non-blocking; logged

</td>
</tr>
</table>

**Execution modes**: use `interactive` mode until the deploy pattern has been validated in the specific customer environment at least twice. Production deployments should not run in `autonomous` mode on a first deployment.

### OTel Observability Middleware Region-Specific Deployment

**OTel Observability Middleware Export Destinations**

<table>
<tr>
<th valign="top">

Environment

</th>
<th valign="top">

Export Destination

</th>
<th valign="top">

Configuration

</th>
</tr>
<tr>
<td valign="top">

Local development

</td>
<td valign="top">

stdout (logging-only)

</td>
<td valign="top">

No configuration needed; graceful fallback

</td>
</tr>
<tr>
<td valign="top">

Standard SAP regions (EU, US East, AP)

</td>
<td valign="top">

SAP Cloud Logging via CF `cloud-logging` binding

</td>
<td valign="top">

Binding auto-detected; no env var needed

</td>
</tr>
<tr>
<td valign="top">

Sovereign regions (CN40, NS2, KSA)

</td>
<td valign="top">

Custom OTLP endpoint

</td>
<td valign="top">

Set `OTEL_EXPORTER_OTLP_ENDPOINT`

</td>
</tr>
<tr>
<td valign="top">

Canary / sandbox

</td>
<td valign="top">

stdout (logging-only)

</td>
<td valign="top">

Cloud Logging unavailable; graceful fallback

</td>
</tr>
</table>

**SAP OTel pipeline infrastructure**: within SAP-managed regions, OTel telemetry exported to SAP Cloud Logging routes through **OctoRoute** (intelligent routing layer), **OctoBus** (event bus), and **CaaS** (Collector-as-a-Service). This infrastructure is transparent to FDE engineers. It becomes relevant when investigating delayed or missing telemetry — the issue may be in the OctoRoute/OctoBus pipeline rather than in the agent's OTel instrumentation.

### Dynatrace CF Integration — Scoped API Token Requirement

When integrating Dynatrace APM via Cloud Foundry service binding, use a scoped API token to avoid 403 errors on the Dynatrace ingest endpoint:

-   Use `cf bind-service` with the Dynatrace service instance
-   Ensure the Dynatrace API token has the **ingest scope** (`metrics.ingest`, `logs.ingest`, `traces.ingest`) — not a full admin token

A mis-scoped token causes silent metric drop: the application starts successfully, but no metrics, logs, or traces appear in Dynatrace. The failure is on the Dynatrace ingest side and requires checking Dynatrace audit logs to diagnose.

**Related Information**  


[Agent Quality Framework](agent-quality-framework-5f15077.md)

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

[Security and Data Protection](security-and-data-protection-a403094.md)

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

[Agent Evaluation](agent-evaluation-bb08bc0.md)

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

[Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

[SAP AI Core](https://help.sap.com/docs/AI_CORE/2d6c5984063c40a59eda62f4a9135bee/d029a32c22fb45fbb607e6a2c48c8a0e.html "SAP AI Core documentation — service plans, model lifecycle, regional availability, and technical requirements.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — agent lifecycle management, governance registration, and production gate for commercial agent deployments.") :arrow_upper_right:
