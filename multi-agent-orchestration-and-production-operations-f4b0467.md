<!-- loiof4b0467846094baebdeb259fb7dc46ce -->

# Multi-Agent Orchestration and Production Operations

This document covers when and how to compose multiple specialist agents, the three orchestration topologies, agent versioning strategy, rollback mechanics, and environment migration requirements. These are production readiness concerns that must be designed in Discovery and tested in Execute — not improvised at handover.



<a name="loiof4b0467846094baebdeb259fb7dc46ce__section_when_multi"/>

## When Multi-Agent Is Appropriate

Multi-agent adds latency, complexity, and more failure points. It is the right choice only when the alternatives are genuinely insufficient:

**Single Agent vs. Multi-Agent Decision Criteria**


<table>
<tr>
<th valign="top">

Situation

</th>
<th valign="top">

Recommendation

</th>
</tr>
<tr>
<td valign="top">

Complex single-domain workflow with many tool calls

</td>
<td valign="top">

Single agent with well-structured prompts and self-verification

</td>
</tr>
<tr>
<td valign="top">

Different data retrieval steps, same authorization scope

</td>
<td valign="top">

Single agent with multiple `sapMcpClient` tools

</td>
</tr>
<tr>
<td valign="top">

Sequential steps where step B always follows step A

</td>
<td valign="top">

Single workflow with explicit nodes; no multi-agent needed

</td>
</tr>
<tr>
<td valign="top">

Parallel sub-tasks requiring genuinely different authorization scopes

</td>
<td valign="top">

Multi-agent: each worker holds only the scope it needs

</td>
</tr>
<tr>
<td valign="top">

Sub-task outputs need independent quality verification before aggregation

</td>
<td valign="top">

Multi-agent: each worker has its own verification layer

</td>
</tr>
<tr>
<td valign="top">

Sub-tasks use genuinely different models \(tabular vs. reasoning\)

</td>
<td valign="top">

Multi-agent: model selection per worker without conflict

</td>
</tr>
</table>

SAP's orchestration hierarchy: **Joule orchestrates → domain agents decompose → specialist agents execute**. The orchestrator is itself an agent registered on the BAIP managed runtime with its own identity, authorization scope, and telemetry. ORD \(Open Resource Discovery\) registration via UMS with the `sap:ai:v1` policy level is normatively required for all agents in a multi-agent topology — agents not ORD-registered are invisible to Agent Gateway.



<a name="loiof4b0467846094baebdeb259fb7dc46ce__section_topologies"/>

## Three Orchestration Topologies

**Topology Comparison**


<table>
<tr>
<th valign="top">

Topology

</th>
<th valign="top">

Structure

</th>
<th valign="top">

Verification Responsibility

</th>
<th valign="top">

When to Use

</th>
</tr>
<tr>
<td valign="top">

1 — Flat \(Orchestrator → N Workers\)

</td>
<td valign="top">

Orchestrator decomposes task, invokes N workers in parallel via `sapAgent(blocking=false)`, aggregates results

</td>
<td valign="top">

Orchestrator holds verification for the aggregated output only

</td>
<td valign="top">

Worker outputs only meaningful in combination; not independently committed to a system of record

</td>
</tr>
<tr>
<td valign="top">

2 — Per-Worker Supervised

</td>
<td valign="top">

Each worker has its own supervisor \(Workflow A inline quality relay\); orchestrator receives only supervisor-approved outputs

</td>
<td valign="top">

Distributed: each supervisor verifies its worker independently; orchestrator applies cross-worker consistency check

</td>
<td valign="top">

Worker outputs are independently significant; each carries regulatory or financial risk on its own

</td>
</tr>
<tr>
<td valign="top">

3 — Shared Supervisor

</td>
<td valign="top">

Single supervisor monitors all workers via shared state store; correlates cross-worker patterns

</td>
<td valign="top">

Centralized: supervisor maintains pending set; atomic commit principle — partial approval is not approval; all workers or none

</td>
<td valign="top">

Workers must collectively satisfy a combined business constraint; partial approval creates compliance exposure

</td>
</tr>
</table>



<a name="loiof4b0467846094baebdeb259fb7dc46ce__section_versioning"/>

## Agent Versioning Strategy

A deployed agent is a coordinated set of five versioned components that must be updated and rolled back as a group:

**Five-Component Version Stack**


<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Where It Is Versioned

</th>
<th valign="top">

Change Risk

</th>
</tr>
<tr>
<td valign="top">

Joule Studio workflow definition

</td>
<td valign="top">

SAP AI Agent Hub / Joule Studio version history

</td>
<td valign="top">

High

</td>
</tr>
<tr>
<td valign="top">

SAP Prompt Registry entry

</td>
<td valign="top">

Git repository + workflow parameter \(current\); SAP Prompt Registry when `promptTemplate` source ships

</td>
<td valign="top">

Medium to High

</td>
</tr>
<tr>
<td valign="top">

Model selection

</td>
<td valign="top">

`sapOrchestration` model parameter \(BAIP configuration\)

</td>
<td valign="top">

High

</td>
</tr>
<tr>
<td valign="top">

Threshold configuration

</td>
<td valign="top">

n8n workflow variables \(`$vars`\) in Joule Studio UI — no redeployment required

</td>
<td valign="top">

Low

</td>
</tr>
<tr>
<td valign="top">

Authorization scope

</td>
<td valign="top">

BAIP application registration \(policy assignments\)

</td>
<td valign="top">

Very High \(requires DPCE review if scope expands\)

</td>
</tr>
</table>



<a name="loiof4b0467846094baebdeb259fb7dc46ce__section_rollback"/>

## Rollback Triggers and Mechanics

A rollback is warranted when production quality metrics show a confirmed regression correlated with a version change:

**Rollback Trigger Signals**


<table>
<tr>
<th valign="top">

Signal

</th>
<th valign="top">

Threshold

</th>
</tr>
<tr>
<td valign="top">

Human override rate increase

</td>
<td valign="top">

\> 20 percentage points above pre-deployment baseline

</td>
</tr>
<tr>
<td valign="top">

Verification rejection rate increase

</td>
<td valign="top">

\> 15 percentage points above pre-deployment baseline

</td>
</tr>
<tr>
<td valign="top">

Process KPI degradation

</td>
<td valign="top">

\> 20% below pre-deployment baseline

</td>
</tr>
<tr>
<td valign="top">

Structured output format failures

</td>
<td valign="top">

\> 5% of invocations producing malformed output

</td>
</tr>
<tr>
<td valign="top">

LLM refusal rate

</td>
<td valign="top">

Any non-zero refusal rate in production \(zero expected baseline\)

</td>
</tr>
</table>

Rollback order: threshold → model → prompt → workflow. In a multi-agent deployment, confirm interface contract compatibility before rolling back individual workers — a schema change in one worker's output can silently break a downstream agent.



<a name="loiof4b0467846094baebdeb259fb7dc46ce__section_environment_migration"/>

## Environment Migration \(Dev → Test → Production\)

Three artifacts must never be copied between environments — they must always be re-provisioned:

1.  **Credentials** — never copy service keys, API credentials, or connection strings from dev to test to production
2.  **Development/test thresholds** — anomaly detection baselines and human approval thresholds must be calibrated per environment from observed behavior; dev thresholds are almost always wrong for production
3.  **Agent identities** — each environment requires a fresh BAIP application registration; re-using dev identities in production creates authorization audit trail gaps

> ### Note:  
> Verify Agent Action Metering \(TR13\) is live before production go-live. TR13 is required for accurate AI Units cost attribution per agent workflow. It is a mandatory pre-production requirement, not a post-launch addition.

**Related Information**  


[Plan and Build — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html)

[Observe and Analyze — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html)

[CIS CoE Operations and Reusable Components](cis-coe-operations-and-reusable-components-363e20e.md)

[Precision Guardrails and Governance](precision-guardrails-and-governance-8f11ea8.md)
