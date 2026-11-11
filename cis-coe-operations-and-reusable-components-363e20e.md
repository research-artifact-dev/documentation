<!-- loio363e20e674284065b64edce196ded336 -->

# CIS CoE Operations and Reusable Components

This document describes the CIS Center of Excellence \(CoE\) Operations Model for managing deployed agents after productization, the two deployment topologies, and the three-layer reusability stack. The CIS CoE model shifts Industry AI delivery from reactive incident response to proactive, continuous lifecycle management.



<a name="loio363e20e674284065b64edce196ded336__section_lifecycle_shift"/>

## From Build-and-Handover to Continuous Lifecycle Management

AI agents do not behave like stable software artifacts after productization. Their quality is a function of the runtime conditions they operate in — the data they receive, the models they use, the processes they support — and all of these change over time.

**Traditional FDE vs. CIS CoE Operations Model**


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



<a name="loio363e20e674284065b64edce196ded336__section_topologies"/>

## Two Deployment Topologies

**Topology A — Agent on Customer Tenant**: the agent runs in the customer's own BAIP sub-account. The customer owns the runtime, data, and observability infrastructure. CIS has no default access. The supervisor agent serves as the autonomous CIS proxy: it performs monitoring and intervention that CIS would otherwise need direct access to do.

**Topology B — Agent on SAP-Managed Tenant**: the agent runs in a SAP-managed BAIP environment as part of a SAP-hosted Industry AI subscription. SAP/CIS is the operator. CIS has full Agent Hub observability across all agents in the managed tenant, direct BAIP Cloud Logging access, and full Automation tier capability.

See [Observe and Analyze — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html) for observability tooling and [Secure and Govern](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/3ea62b45d7f64a8aab15321424ffb37c.html) for the governance layer.



<a name="loio363e20e674284065b64edce196ded336__section_operating_rhythm"/>

## CIS CoE Operating Rhythm

**Four Severity Tiers**


<table>
<tr>
<th valign="top">

Severity

</th>
<th valign="top">

Definition

</th>
<th valign="top">

Response Target

</th>
</tr>
<tr>
<td valign="top">

Critical

</td>
<td valign="top">

Agent blocked; no outputs being committed; immediate business impact

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

Quality regression confirmed \(rollback triggers met\); degraded but operating

</td>
<td valign="top">

2 business days

</td>
</tr>
<tr>
<td valign="top">

Medium

</td>
<td valign="top">

Anomaly detected; pattern under analysis; no confirmed quality impact yet

</td>
<td valign="top">

5 business days

</td>
</tr>
<tr>
<td valign="top">

Low

</td>
<td valign="top">

Advisory finding; threshold drift; improvement opportunity identified

</td>
<td valign="top">

Next operating rhythm cycle

</td>
</tr>
</table>



<a name="loio363e20e674284065b64edce196ded336__section_project_lightspeed"/>

## Project LightSpeed — CIS Standard FDE Delivery Toolchain

Project LightSpeed is the CIS-built delivery toolchain for FDE engagements — a 13-stage pipeline that transforms raw customer discovery artifacts \(workshop notes, CDS models, OData specs, meeting transcripts\) into a deployed, observable, Joule-connected multi-agent application.

**Project LightSpeed Two-Phase Delivery Model**


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

Compiled knowledge artifact + agent topology design + readiness scores + coaching questions

</td>
</tr>
<tr>
<td valign="top">

Phase 2 — Enterprise Integration and Production PoC

</td>
<td valign="top">

Stages 9–13: spec generation, agent scaffolding, Joule capability registration, evaluation scorecard, BTP provisioning

</td>
<td valign="top">

Weeks \(across the FDE Execute sprint\)

</td>
<td valign="top">

Deployed agents with Joule surface, OTel observability, BTP infrastructure

</td>
</tr>
</table>



<a name="loio363e20e674284065b64edce196ded336__section_reusability_stack"/>

## Three-Layer Reusability Stack

**Reusability Layer Summary**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

Definition

</th>
<th valign="top">

FDE Effort

</th>
<th valign="top">

Maintenance Owner

</th>
</tr>
<tr>
<td valign="top">

Layer 1 — Deploy As-Is

</td>
<td valign="top">

Zero per-engagement content. Domain-agnostic components that operate identically in every engagement: Supervisor Workflow C, audit trail base schema, sanitization rules, version stack discipline, supervisor fail-safe rule

</td>
<td valign="top">

Zero — inherited from CIS Starter Kit unchanged

</td>
<td valign="top">

CIS T&O

</td>
</tr>
<tr>
<td valign="top">

Layer 2 — Parameterize and Deploy

</td>
<td valign="top">

Fixed skeleton; per-engagement content in defined slots. Structure and routing logic are the same in every engagement; threshold values, prompts, and verification rules vary: Supervisor Workflow A and B skeletons, guardrail stack structure, eval test suite structure

</td>
<td valign="top">

Configuration and calibration only — no structural design

</td>
<td valign="top">

CIS T&O \(skeleton\); FDE team \(configuration\)

</td>
</tr>
<tr>
<td valign="top">

Layer 3 — Engagement-Specific

</td>
<td valign="top">

Cannot be reused. Worker agent business logic, process KPIs, Workflow A verification rules, LLM critic system prompts, human approval threshold values, authorization matrices, anomaly detection thresholds

</td>
<td valign="top">

Full design and build work required

</td>
<td valign="top">

FDE team

</td>
</tr>
</table>

> ### Note:  
> The CIS Starter Kit described in this concept is a specification of what should be built, not a set of artifacts that currently exists. Teams reading this section should treat it as a design target and delivery roadmap, not a download.

**Related Information**  


[SAP AI Agent Hub — Step-by-Step Guide](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/5938fa55ea1241dc8d786764aa610f4a.html)

[Optimize and Decommission — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/38105cdd332547d6b0877a647bcd0fbe.html)

[Multi-Agent Orchestration and Production Operations](multi-agent-orchestration-and-production-operations-f4b0467.md)

[Customer Enablement and Adoption](customer-enablement-and-adoption-7abe271.md)
