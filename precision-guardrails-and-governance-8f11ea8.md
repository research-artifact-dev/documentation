<!-- loio8f11ea852932452caacc93fea508937b -->

# Precision Guardrails and Governance

Precision guardrails are the design layer that intercepts numerical and business-critical outputs before they become committed business facts. They are not generic output validation — they are a purpose-built protection stack for high-stakes quantitative decisions where LLM errors can create regulatory, financial, or operational consequences.



<a name="loio8f11ea852932452caacc93fea508937b__section_why_numerical"/>

## Why Numerical Outputs Require Special Handling

LLMs are not calculators. They produce numerically plausible outputs — numbers that look correct and are formatted appropriately — but plausibility is not accuracy. Errors cluster around systematic patterns that a deterministic reference check can reliably detect:

**Numerical Output Failure Modes by Industry AI Domain**


<table>
<tr>
<th valign="top">

Industry AI Domain

</th>
<th valign="top">

Numerical Output

</th>
<th valign="top">

Failure Mode

</th>
<th valign="top">

Consequence

</th>
</tr>
<tr>
<td valign="top">

Autonomous Regulated Manufacturing

</td>
<td valign="top">

Batch release quantity \(kg, units, volume\)

</td>
<td valign="top">

Unit confusion: kg vs. metric ton

</td>
<td valign="top">

Regulatory deviation; incorrect shipment; potential recall

</td>
</tr>
<tr>
<td valign="top">

Autonomous Commodity Management

</td>
<td valign="top">

Hedge ratio

</td>
<td valign="top">

Sign error: long/short inversion

</td>
<td valign="top">

Unintended directional exposure; financial loss

</td>
</tr>
<tr>
<td valign="top">

Autonomous Revenue Growth Management

</td>
<td valign="top">

Pricing recommendation

</td>
<td valign="top">

Currency unit confusion \(EUR vs. USD\)

</td>
<td valign="top">

Systematic under- or over-pricing

</td>
</tr>
<tr>
<td valign="top">

Autonomous Asset Management

</td>
<td valign="top">

Maintenance cost estimate

</td>
<td valign="top">

Wrong order of magnitude

</td>
<td valign="top">

Budget overrun; deferred maintenance misclassification

</td>
</tr>
<tr>
<td valign="top">

Autonomous Adaptive Production

</td>
<td valign="top">

Production order quantity

</td>
<td valign="top">

Integer vs. float: 1.5 batches vs. 1 or 2

</td>
<td valign="top">

Schedule disruption; material over-ordering

</td>
</tr>
</table>

**Design priority**: if a deterministic SAP service can compute the answer authoritatively, the agent should call that service as a tool and use its output — not ask the LLM to reason to the same number.



<a name="loio8f11ea852932452caacc93fea508937b__section_fail_warn"/>

## FAIL and WARN Guardrail Classification

Guardrail checks use a two-tier classification that governs whether a failing check blocks the operation or flags it for review:

**Guardrail Classification**


<table>
<tr>
<th valign="top">

Classification

</th>
<th valign="top">

Behavior

</th>
<th valign="top">

Used When

</th>
</tr>
<tr>
<td valign="top">

FAIL

</td>
<td valign="top">

Blocking — the operation is halted; the issue must be resolved before proceeding

</td>
<td valign="top">

Proceeding would cause incorrect behavior, security failure, or compliance violation

</td>
</tr>
<tr>
<td valign="top">

WARN

</td>
<td valign="top">

Advisory — the operation may proceed; the finding is logged and flagged for review

</td>
<td valign="top">

The condition is sub-optimal but does not prevent correct operation

</td>
</tr>
</table>

Example FAIL conditions: credential secret not in the same namespace as the consuming pod; `spec_sha256` mismatch; Joule capability config schema violation; credential file not in `.gitignore`.

Example WARN conditions: routing description below quality threshold; test coverage below target.



<a name="loio8f11ea852932452caacc93fea508937b__section_guardrail_stack"/>

## The Four-Layer Guardrail Stack

Precision guardrails operate as a four-layer stack. Each layer addresses a different point in the agent's decision process and a different failure mode:

**Four-Layer Guardrail Stack**


<table>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

What It Does

</th>
<th valign="top">

Implementation in Joule Studio

</th>
</tr>
<tr>
<td valign="top">

Layer 1 — Pre-Check

</td>
<td valign="top">

Agent receives only valid input space: `sapMcpClient` retrieves authoritative constraints before reasoning begins

</td>
<td valign="top">

`CUSTOM.sapMcpClient` → SAP pricing engine, batch classification, MRP, valuation model

</td>
</tr>
<tr>
<td valign="top">

Layer 2 — Output Validation

</td>
<td valign="top">

Agent's numerical output verified before it can reach write-back: deterministic Code node check + optional LLM critic

</td>
<td valign="top">

Code node \(range check, unit check, tolerance check\) + second `CUSTOM.sapOrchestration` critic node

</td>
</tr>
<tr>
<td valign="top">

Layer 3 — Human Approval Threshold

</td>
<td valign="top">

Outputs above a defined value or risk threshold require human sign-off before write-back

</td>
<td valign="top">

[SAP Human in the Loop](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/bc403ba6bc114733929e034d9bb3dad4.html) node \(`CUSTOM.sapTaskCenter`\)

</td>
</tr>
<tr>
<td valign="top">

Layer 4 — Audit Trail

</td>
<td valign="top">

Append-only record of every decision: what was known, what was considered, what was chosen, what resulted

</td>
<td valign="top">

16-field schema written to SAP HANA via `sapMcpClient`; no update or delete method

</td>
</tr>
</table>



<a name="loio8f11ea852932452caacc93fea508937b__section_audit_trail"/>

## Audit Trail Schema

The audit trail is append-only by construction. The base schema includes 16 fields:

**Audit Trail Base Fields**


<table>
<tr>
<th valign="top">

Field

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

`timestamp`

</td>
<td valign="top">

ISO 8601 timestamp of the decision event

</td>
</tr>
<tr>
<td valign="top">

`correlation_id`

</td>
<td valign="top">

Unique identifier linking all events in one invocation chain

</td>
</tr>
<tr>
<td valign="top">

`agent_id`

</td>
<td valign="top">

Registered agent identity from BAIP

</td>
</tr>
<tr>
<td valign="top">

`agent_version`

</td>
<td valign="top">

Five-component version stack identifier \(workflow, prompt, model, thresholds, authorization scope\)

</td>
</tr>
<tr>
<td valign="top">

`input_state`

</td>
<td valign="top">

Snapshot of authoritative data retrieved in Layer 1 at decision time

</td>
</tr>
<tr>
<td valign="top">

`agent_output`

</td>
<td valign="top">

Raw output from the agent before guardrail evaluation

</td>
</tr>
<tr>
<td valign="top">

`stage_verdicts`

</td>
<td valign="top">

PASS/FAIL/WARN result from each guardrail layer

</td>
</tr>
<tr>
<td valign="top">

`human_decision`

</td>
<td valign="top">

Approve / Reject / Modify decision from Task Center \(null if no human gate triggered\)

</td>
</tr>
<tr>
<td valign="top">

`human_modified_value`

</td>
<td valign="top">

Value entered by the human if the output was modified \(null otherwise\)

</td>
</tr>
<tr>
<td valign="top">

`committed_value`

</td>
<td valign="top">

Final value written to the system of record

</td>
</tr>
<tr>
<td valign="top">

`delta`

</td>
<td valign="top">

Percentage deviation between agent output and committed value; primary signal for override rate monitoring

</td>
</tr>
</table>

> ### Note:  
> Source data quality is a prerequisite for guardrail effectiveness. If the source system contains incorrect master data, the agent will produce an output that is internally consistent with the bad input it received and may pass all four guardrail layers. Identify the `sapMcpClient` calls that retrieve authoritative constraints during Discovery and confirm data currency with the process owner.

**Related Information**  


[SAP Human in the Loop](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/bc403ba6bc114733929e034d9bb3dad4.html)

[Security and Data Protection](security-and-data-protection-98058b5.md)

[Multi-Agent Orchestration and Production Operations](multi-agent-orchestration-and-production-operations-f4b0467.md)

[Agent Quality Framework](agent-quality-framework-af194ba.md)
