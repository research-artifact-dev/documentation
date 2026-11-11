<!-- loioaf194ba827fa4ca8b2e1b17117c16765 -->

# Agent Quality Framework

The Agent Quality Framework defines how to measure, ensure, and systematically improve the quality of AI agents across the full FDE delivery lifecycle — from Discovery through productization and post-deployment monitoring. Quality must be measured at three levels simultaneously: how the agent reasons, how it uses tools, and what it ultimately does to the business.



<a name="loioaf194ba827fa4ca8b2e1b17117c16765__section_dimensions"/>

## The Three Evaluation Dimensions

SAP's AI-Native North Star Architecture defines agent quality across three dimensions that must be evaluated together. Optimizing for any single dimension in isolation produces misleading results — a measure that becomes a target ceases to be a good measure \(Goodhart's Law\).

**Three Quality Evaluation Dimensions**


<table>
<tr>
<th valign="top">

Dimension

</th>
<th valign="top">

What It Measures

</th>
<th valign="top">

Why Single-Dimension Optimization Fails

</th>
</tr>
<tr>
<td valign="top">

1 — Technical

</td>
<td valign="top">

Goal completion rate, tool call efficiency, self-verification results, hallucination rate, retry/recovery rate

</td>
<td valign="top">

An agent can complete a goal incorrectly; efficient agents may skip validation steps

</td>
</tr>
<tr>
<td valign="top">

2 — Process KPIs

</td>
<td valign="top">

Operational metrics tied to the business process being automated — defined per Industry AI domain in Discovery

</td>
<td valign="top">

High technical scores on a poorly specified goal produce no business value

</td>
</tr>
<tr>
<td valign="top">

3 — Business Outcomes

</td>
<td valign="top">

Financial and operational results: consumed ACV, cost reduction, exception reduction, throughput improvement

</td>
<td valign="top">

Most lagging indicator — appears weeks or months after deployment; justifies the value-based contract

</td>
</tr>
</table>

**Success rate** is the primary release gate across all three dimensions. A test case's success rate is binary: 1 if every defined validation passes, 0 if any single validation fails. This binary definition forces the team to enumerate the exact conditions an agent must satisfy rather than averaging over partial correctness. The SAP aeval framework \(TR9 — mandatory offline and online evaluation for all production agents\) uses success rate as its primary aggregation metric.



<a name="loioaf194ba827fa4ca8b2e1b17117c16765__section_readiness_gates"/>

## Pre-Build Readiness Gates \(G1–G4\)

Before generating an agent specification, the extracted knowledge from the Discovery phase must satisfy four readiness gates. These gates prevent the most common cause of wasted build effort: generating agent code from discovery outputs that are structurally incomplete.

**Pre-Build Readiness Gate Scoring**


<table>
<tr>
<th valign="top">

Gate

</th>
<th valign="top">

What It Checks

</th>
<th valign="top">

Pass Threshold

</th>
</tr>
<tr>
<td valign="top">

G1 — Entity Completeness

</td>
<td valign="top">

Proportion of extracted entities resolved against SAP canonical catalog \(EKX or offline S/4 catalog\)

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G2 — Goal Coverage

</td>
<td valign="top">

Every extracted Capability has a parent Goal

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G3 — Actor-Process Binding

</td>
<td valign="top">

Every extracted Process has an assigned Actor

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
<tr>
<td valign="top">

G4 — Integration Readiness

</td>
<td valign="top">

Every Integration has an API binding

</td>
<td valign="top">

≥ 0.8

</td>
</tr>
</table>

When a gate score falls between 0.5 and 0.8, the system auto-generates coaching questions — targeted gap-filling queries designed to be taken back to the customer in the next working session. A score below 0.5 is a hard stop: insufficient grounding for reliable agent design.



<a name="loioaf194ba827fa4ca8b2e1b17117c16765__section_build_quality"/>

## Build-Time Quality

Build-time quality covers practices and tooling applied before an agent is released to any environment.

### Eval Design in Joule Studio

Every agent built in [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html) has an associated evaluation configuration. The eval design must answer:

1.  **What is the agent supposed to achieve?** — Define the goal state as a machine-checkable condition, not a vague description.
2.  **What are the boundary conditions?** — Define cases where the agent must NOT act \(transaction value above approval threshold, output confidence below threshold\). These are the guardrail test cases.
3.  **What is the test data strategy?** — A legal question as much as a technical one. Build-time evals must work with synthetic data so they can run continuously under a TEA \(Test and Evaluation Agreement\) without legal friction.
4.  **What is acceptable failure behavior?** — Define what the agent does when it cannot complete the goal. Undefined failure behavior is the single most common source of production incidents.

### Post-Build Evaluation Scorecard

After agent code is generated and before deployment, a 6-category scorecard validates readiness:

**Post-Build Evaluation Categories**


<table>
<tr>
<th valign="top">

Category

</th>
<th valign="top">

What It Checks

</th>
<th valign="top">

Method

</th>
</tr>
<tr>
<td valign="top">

A — Spec integrity

</td>
<td valign="top">

`agent.spec.json` is valid, complete, and internally consistent

</td>
<td valign="top">

Deterministic

</td>
</tr>
<tr>
<td valign="top">

B — Generation drift

</td>
<td valign="top">

`spec_sha256` in generated code matches the source spec

</td>
<td valign="top">

Deterministic

</td>
</tr>
<tr>
<td valign="top">

C — Joule config

</td>
<td valign="top">

Joule capability bundle structure, naming invariants, schema validation

</td>
<td valign="top">

Deterministic

</td>
</tr>
<tr>
<td valign="top">

D — Routing description quality

</td>
<td valign="top">

Each agent's routing description is distinctive, domain-specific, and does not overlap with other agents

</td>
<td valign="top">

LLM-graded

</td>
</tr>
<tr>
<td valign="top">

E — MUST-rule compliance

</td>
<td valign="top">

Business constraints encoded as MUST rules in `golden_scenarios.yaml` are satisfied

</td>
<td valign="top">

LLM-graded

</td>
</tr>
<tr>
<td valign="top">

F — Coverage and completeness

</td>
<td valign="top">

All required capabilities have corresponding agent behaviors

</td>
<td valign="top">

LLM-graded

</td>
</tr>
</table>

Category B — generation drift — is the most operationally critical check. A `spec_sha256` mismatch is a blocking failure: the deployment is invalid regardless of whether other categories pass.



<a name="loioaf194ba827fa4ca8b2e1b17117c16765__section_runtime_quality"/>

## Runtime Quality and the Quality Flywheel

Runtime quality monitoring operates through the SAP aeval framework's online evaluation path \(TR9 mandatory\): sampled production traffic is scored automatically via the Data Warehousing Cloud \(DwC\) pipeline, deployed by the App Foundation platform.

The Quality Flywheel maps evaluation activities to the 12-week FDE sprint:

-   **Weeks 1–2**: define test cases in aeval BDD format \(given/when/then\); establish golden dataset; set success rate threshold
-   **Weeks 3–6**: build agent and supervisor; run offline eval in CI on every commit; success rate trend is the sprint health indicator
-   **Weeks 7–9**: EAC launch; switch online eval to live traffic; calibrate anomaly detection baselines
-   **Weeks 10–12**: productization gate; all TR1–TR14 checks must pass; online eval must hold success rate target



<a name="loioaf194ba827fa4ca8b2e1b17117c16765__section_escalation"/>

## Runtime Escalation Path

The escalation path operates across four levels:

-   **Level 0 — Automated**: agent self-verifies and commits normally
-   **Level 1 — Monitored**: output flagged for async review; committed but logged for pattern analysis
-   **Level 2 — Supervised**: supervisor intercepts output; routes to `sapTaskCenter` for human sign-off before write-back
-   **Level 3 — Blocked**: supervisor blocks the output; creates an incident ticket; no write-back occurs

See [Task Management in Joule Work](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/8b0fa61be9d14d33a6b8ceec4c1c403c.html) for the human-in-the-loop implementation.

**Related Information**  


[Joule Studio — SAP-Specific Nodes](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/e2fbd2fd7fda4fceadb7cf2ef4b22530.html)

[Observe and Analyze — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html)

[Precision Guardrails and Governance](precision-guardrails-and-governance-8f11ea8.md)

[System Architecture and Build Paths](system-architecture-and-build-paths-a6daf78.md)
