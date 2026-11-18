<!-- loiof940c93e5f5a46a99a5dc358e66d4325 -->

# Agent Evaluation in FDE Sprints

This topic defines how to measure agent quality in FDE sprints — the evaluation mechanism, tooling, pre-build readiness gates, the `aeval` framework, the post-deployment feedback loop, and sprint timing requirements for TR9 compliance.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_why_eval_chapter"/>

## Why Evaluation Requires Its Own Framework

Evaluation in an FDE context has properties that make informal approaches fail:

-   **Agents are non-deterministic.** The same input can produce different outputs on different runs. A single manual test run is not quality evidence; a structured test suite run across multiple iterations is.
-   **TR9 is mandatory.** SAP's AI Golden Path requires both offline evaluation (scripted test suites in CI) and online evaluation (monitoring of live production sessions) for every production agent. This is not a recommendation.
-   **Evaluation must start early.** FDE sprints are short (target: 90 days). An evaluation infrastructure built in week 9 cannot generate the quality evidence needed for the FDE Gate at week 6 or the EAC launch at week 9.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_eval_lifecycle"/>

## Three-Phase Evaluation Lifecycle

Evaluation in FDE sprints spans three distinct phases, each with its own tools, owners, and purpose.

**Evaluation Lifecycle Phases**

<table>
<thead>
<tr>
<th valign="top">Phase</th>
<th valign="top">When</th>
<th valign="top">Tool</th>
<th valign="top">Purpose</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Pre-build

</td>
<td valign="top">

Before agent spec is generated

</td>
<td valign="top">

G1–G4 Readiness Gates

</td>
<td valign="top">

Verify discovery outputs are complete enough to build from

</td>
</tr>
<tr>
<td valign="top">

Spec quality

</td>
<td valign="top">

End of Discovery; pre-EAC

</td>
<td valign="top">

`agentic-eval` skill (11+1 criteria)

</td>
<td valign="top">

Verify spec is complete and AI-ready; verify code-spec alignment before customer go-live

</td>
</tr>
<tr>
<td valign="top">

Post-build

</td>
<td valign="top">

After agent code is generated, before deploy

</td>
<td valign="top">

6-category eval scorecard

</td>
<td valign="top">

Verify agent spec, Joule config, routing quality, and compliance with MUST rules

</td>
</tr>
<tr>
<td valign="top">

Post-deployment

</td>
<td valign="top">

Continuously in production

</td>
<td valign="top">

Two-path feedback loop + `aeval` online eval

</td>
<td valign="top">

Detect regressions, close learning loops, improve agent over time

</td>
</tr>
</tbody>
</table>

The `aeval` offline/online framework covers the CI-gated release gate between post-build and post-deployment. The G1–G4 gates and the 6-category scorecard extend evaluation earlier — into the discovery and code-generation phases — where defects are cheapest to fix. The `agentic-eval` skill adds a fourth dimension: structured spec-and-build quality measurement at the two sprint points where spec drift is most likely to surface.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_g1_g4_gates"/>

## Pre-Build Readiness Gates (G1–G4)

Before generating an agent spec, the extracted knowledge must meet four readiness gates. These gates prevent the most common cause of wasted build effort: generating agent code from discovery outputs that are structurally incomplete. Each gate scores 0–1:

-   **≥ 0.8** — proceed
-   **≥ 0.5 and < 0.8** — proceed with gaps; coaching questions auto-generated
-   **< 0.5** — stop and return to discovery

**G1–G4 Readiness Gates**

<table>
<thead>
<tr>
<th valign="top">Gate</th>
<th valign="top">What It Checks</th>
<th valign="top">Pass Threshold</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

G1 — Entity Completeness

</td>
<td valign="top">

Proportion of entities successfully resolved against SAP's EKX/canonical catalog

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
</tbody>
</table>

When a gate score falls between 0.5 and 0.8, the system auto-generates **coaching questions** — targeted gap-filling queries designed to be taken back to the customer in the next working session. This transforms discovery gaps from silent blockers into structured follow-up items.

The 0.5 hard stop is deliberate: generating an agent spec from a foundation that is less than half-resolved against SAP's canonical catalog produces agent code that cannot correctly route to the right APIs and will fail the post-build scoring categories A, C, and F before any functionality is tested.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-synthesize` skill.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_post_build_scorecard"/>

## Post-Build Evaluation Scorecard

After agent code is generated and before deployment, a 6-category scorecard validates readiness. Categories A–C are deterministic checks that execute in milliseconds; categories D–F are LLM-graded. Output: `evals/eval-report.md`.

**6-Category Post-Build Scorecard**

<table>
<thead>
<tr>
<th valign="top">Category</th>
<th valign="top">What It Checks</th>
<th valign="top">Method</th>
</tr>
</thead>
<tbody>
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

Joule capability bundle structure, naming invariants, schema validation via Joule MCP

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

Each agent's routing description is distinctive, domain-specific, contains no personal pronouns, and does not overlap with other agents

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

Business constraints encoded as MUST rules (from `golden_scenarios.yaml`) are satisfied

</td>
<td valign="top">

LLM-graded

</td>
</tr>
<tr>
<td valign="top">

F — Coverage / completeness

</td>
<td valign="top">

All required capabilities have corresponding agent behaviors

</td>
<td valign="top">

LLM-graded

</td>
</tr>
</tbody>
</table>

**Category B — generation drift** is the most operationally critical check. If `agent.spec.json` is edited after code generation — common in iterative FDE development — the generated agent no longer implements the spec it claims to implement. A `spec_sha256` mismatch is a blocking failure: the deployment is invalid regardless of whether the other categories pass. The fix is to re-run code generation from the updated spec, then re-evaluate.

**Category D — routing description quality** addresses a failure mode that is invisible until production: two agents with overlapping or generic routing descriptions cause the Joule orchestrator to misroute user requests. The LLM-graded check compares all routing descriptions in the bundle against each other and flags overlap risk before deployment.

**MUST-rule compliance (Category E)** connects to [Precision Guardrails](precision-guardrails-0beada2.md): the MUST rules encoded in `golden_scenarios.yaml` are the machine-verifiable form of the business constraints that the guardrail design was built to enforce.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-evals` skill.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_feedback_loop"/>

## Post-Deployment Feedback Loop

The feedback loop captures every HITL override event and transforms it into a learning signal. It runs on two parallel paths:

**Spec path (human-gated):** HITL override → analyst LLM proposes targeted edit to `agent.spec.json` → human approves → redeploy triggered. This path fires only when the spec genuinely needs to change.

**Memory path (autonomous):** Per-conversation traces → analyst LLM proposes memory changes → auditor LLM confirms or rejects → ACTIVE memories are injected into the agent's system prompt on subsequent turns. No human gate for memory updates; the auditor LLM provides the quality check.

Both paths handle the same four learning cases:

**Four Learning Cases**

<table>
<thead>
<tr>
<th valign="top">Case</th>
<th valign="top">What Happened</th>
<th valign="top">Action</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Case 1 — Agent wrong + human overrode correctly

</td>
<td valign="top">

Agent made an error; human catch was correct

</td>
<td valign="top">

Spec path: propose targeted spec fix. Memory path: add corrective memory.

</td>
</tr>
<tr>
<td valign="top">

Case 2 — Both wrong

</td>
<td valign="top">

Agent made an error; human override was also incorrect

</td>
<td valign="top">

No spec change. Flag for human review. Do NOT propagate the override.

</td>
</tr>
<tr>
<td valign="top">

Case 3 — Agent correct + human overrode incorrectly

</td>
<td valign="top">

Agent was right; human override introduced an error

</td>
<td valign="top">

**Critical: do NOT propose a spec change.** Flag the override as an operator training issue. Track separately.

</td>
</tr>
<tr>
<td valign="top">

Case 4 — Agent correct + human agreed, outcome confirmed

</td>
<td valign="top">

Agent performed correctly; outcome confirmed good

</td>
<td valign="top">

Reinforce via memory path. No spec change needed.

</td>
</tr>
</tbody>
</table>

**Case 3 — the operator training issue** is the most subtle and the most consequential. Without explicit handling of this case, a feedback system learns incorrect behaviors from operator mistakes. Every implementation of HITL feedback must handle Case 3 explicitly, with a separate tracking path that feeds into operator training, not agent spec revision.

When the analyst LLM determines Case 3 (agent correct, human wrong), it logs the override event with a `case_3` flag. These flagged events are surfaced to the engagement's Adoption Lead as a distinct metric — **"agent correct, human override rate"** — separate from the general HITL override rate. A rising `agent correct, human override rate` is an adoption signal, not a quality signal; it should trigger an operator training review, not a prompt revision.

**Deferred Outcome Tracking:** Not all outcomes are immediately observable. The feedback loop supports configurable outcome horizons: the system waits for a downstream outcome event before closing the learning loop. Outcomes that return `outcome_confirmed_bad` escalate the associated memory or spec change to CRITICAL severity, requiring human review before any update is applied.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-feedback` skill.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_testing_vs_eval"/>

## Testing vs. Evaluation — Two Distinct Activities

SAP's AI Golden Path draws a deliberate distinction:

**Testing vs. Evaluation**

<table>
<thead>
<tr>
<th valign="top">Activity</th>
<th valign="top">Purpose</th>
<th valign="top">When</th>
<th valign="top">Owner</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Testing

</td>
<td valign="top">

Ongoing defect finding throughout development

</td>
<td valign="top">

Continuously, throughout Development

</td>
<td valign="top">

FDE Application Engineers

</td>
</tr>
<tr>
<td valign="top">

Evaluation

</td>
<td valign="top">

Milestone-based comparison: has the agent improved between versions?

</td>
<td valign="top">

At sprint milestones (Week 6 MVP, Week 9 pre-deployment, each productization release)

</td>
<td valign="top">

FDE Dev Architect + Domain PM

</td>
</tr>
</tbody>
</table>

Both are required. Testing uses unit tests, integration tests, and smoke tests. Evaluation uses structured test suites that measure agent behavior on representative user journeys across multiple iterations.

The distinction also maps to tools: testing uses standard test frameworks (pytest, jest); evaluation uses SAP's `aeval` framework.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_aeval_framework"/>

## The aeval Framework

### What It Is

`aeval` is SAP's internal Python framework and CLI for evaluating AI agents. It implements the two evaluation modes required by TR9:

-   **Offline mode** (`aeval run`): scripted test cases run against a live or local agent. Results exported as JSON and HTML reports. Gates release decisions.
-   **Online mode** (`aeval-online` / AI Core CronJob): real production traces retrieved from MLflow and scored by an LLM judge. Results written back to MLflow as session-level feedback. Tracks production quality trends continuously.

Both modes use the same evaluation core: SAP's `agent-inspect` library, which invokes 3 independent LLM judge calls per validation (majority vote) via SAP AI Core GenAI Hub. The 3-judge configuration was determined empirically — it achieves reliability close to that of a 5-judge ensemble while materially reducing latency and cost.

### Provisioning aeval on FDE Engagements

`aeval` is available from the `developer-efficiency-tools` repository (`github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools`).

```bash
npx skills add aeval-set-up  # installs the setup skill
# Then follow the /aeval-set-up instructions to configure:
# - GenAI Hub connection (required for LLM judge calls)
# - MLflow instance (required for online eval trace storage)
# - AI Core Extended plan (required for aeval-online CronJob)
```

**Configuration requirements:**

-   **GenAI Hub connection**: the LLM judge calls require a GenAI Hub deployment accessible from the eval environment.
-   **MLflow setup**: offline eval results are stored in MLflow. The LightSpeed-provided MLflow instance (`references/observe-and-eval/`) is the standard starting point.
-   **Sovereign regions**: `aeval-online` requires a GenAI Hub endpoint within the sovereign boundary for judge calls. In CN40, NS2, and KSA, confirm that a GenAI Hub deployment is available in the sovereign region before committing to online eval in production.

### Tracing Modes

**Eval Tracing Modes**

<table>
<thead>
<tr>
<th valign="top">Tracing Mode</th>
<th valign="top">What It Requires</th>
<th valign="top">Metrics Available</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Traceless

</td>
<td valign="top">

Nothing — no OTel setup needed

</td>
<td valign="top">

`output_correctness`, `rule_compliance`, `success_rate`

</td>
</tr>
<tr>
<td valign="top">

Trace-enabled

</td>
<td valign="top">

OTel-compliant traces from BAF or MLflow

</td>
<td valign="top">

All traceless metrics + `tool_call_correctness`, `latency_seconds`, `total_tokens`, `total_cost_usd`, `number_of_tool_calls`

</td>
</tr>
<tr>
<td valign="top">

Online eval

</td>
<td valign="top">

Production traces in MLflow

</td>
<td valign="top">

Per-session `success_rate`, `token_count`, `latency` — continuous monitoring

</td>
</tr>
</tbody>
</table>

**Traceless mode is deliberately available for early-sprint use.** An agent without full OTel instrumentation can still be evaluated for output correctness and rule compliance from sprint week 1. This means evaluation starts before instrumentation is complete.

### OTel Score Schema

The `aeval` OTel output uses `gen_ai.evaluation.*` attributes.

**OTel Score Schema Behavior**

<table>
<thead>
<tr>
<th valign="top">Score Type</th>
<th valign="top">OTel Behavior</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Numeric scores (float, e.g., 0.85)

</td>
<td valign="top">

Written to `gen_ai.evaluation.score.value` (double)

</td>
</tr>
<tr>
<td valign="top">

Label/categorical scores (string, e.g., "high")

</td>
<td valign="top">

Written to `gen_ai.evaluation.score.label` — **not** to `score.value`. Do not attempt to read categorical results from `score.value`.

</td>
</tr>
<tr>
<td valign="top">

Boolean scores

</td>
<td valign="top">

Both `score.value: 1.0|0.0` AND `score.label: "true"|"false"` emitted simultaneously

</td>
</tr>
<tr>
<td valign="top">

NOT_APPLICABLE scores

</td>
<td valign="top">

**Not emitted at all.** Absence in the trace is the signal — a check that did not apply to a session produces no OTel event. Do not equate "no record" with "score = 0".

</td>
</tr>
<tr>
<td valign="top">

Error-skipped scores

</td>
<td valign="top">

`error.type` attribute set to one of: `JUDGE_FAILED`, `AGGREGATION_FAILED`, `TRACE_UNAVAILABLE`, `UNMAPPED_LABEL`. `score.value` and `score.label` are omitted.

</td>
</tr>
</tbody>
</table>

**Dashboard implication**: aggregate success-rate calculations must explicitly handle absent records (NOT_APPLICABLE) vs. zero-scored records (Incomplete). A query that treats missing rows as 0 will silently under-report success rates for checks with frequent NA cases.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_test_case"/>

## The Test Case: the Primary Evaluation Artifact

For each FDE sprint agent, the test case file is a sprint deliverable at the same level as the Joule Studio workflow. It should be version-controlled alongside the agent and reviewed at sprint milestones.

### Test Case DSL

Test cases are YAML files. Each test case has one or more steps; each step has an input message and a set of validations:

```yaml
id: batch_release_recommendation
description: Agent recommends correct batch release quantity
tags: [smoke, correctness, regulated-manufacturing]

test_steps:
  - type: fixed_message
    input_message: "Recommend batch release quantity for production order 500012345"
    agent_response_validations:
      - check: "Agent states a specific quantity with a unit of measure"
      - check: "Agent references the approved batch classification service result"
      - check: "Agent does not recommend a quantity outside the valid range"
    tool_validations:
      expected_tool_calls:
        - tool: get_production_order_constraints
          parameters:
            order_id: { value: "500012345" }
        - tool: get_batch_classification_result
          output:
            check: "Returns a valid batch classification with an approved quantity"
```

### Two Step Types

**Eval Step Types**

<table>
<thead>
<tr>
<th valign="top">Step Type</th>
<th valign="top">When to Use</th>
<th valign="top">What It Tests</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

`fixed_message`

</td>
<td valign="top">

Scripted happy-path and boundary tests; known inputs

</td>
<td valign="top">

Output correctness, tool call compliance, rule adherence on deterministic inputs

</td>
</tr>
<tr>
<td valign="top">

`dynamic_conversation`

</td>
<td valign="top">

Multi-turn workflows where user wording is variable

</td>
<td valign="top">

Agent's ability to guide a conversation to a goal across natural language variation

</td>
</tr>
</tbody>
</table>

For dynamic conversations, a **User Agent** (LLM-powered) drives the conversation from a `task_summary`. The agent under test must reach the defined termination conditions within `max_turns` (maximum: 15).

### What Makes a Good Check

**Check Quality Examples**

<table>
<thead>
<tr>
<th valign="top">Check Quality</th>
<th valign="top">Example</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Vague — avoid

</td>
<td valign="top">

"Agent responds helpfully about the batch"

</td>
</tr>
<tr>
<td valign="top">

Specific — preferred

</td>
<td valign="top">

"Agent states a quantity in kg within the range 800–1200 for this production order"

</td>
</tr>
<tr>
<td valign="top">

Tool-level

</td>
<td valign="top">

"Agent calls `get_batch_classification_result` with `order_id = '500012345'`"

</td>
</tr>
<tr>
<td valign="top">

Safety/guardrail

</td>
<td valign="top">

"Agent does not commit a quantity to the system of record without triggering the Layer 3 approval check"

</td>
</tr>
</tbody>
</table>

The rule of thumb: a check should be falsifiable. If it is impossible to imagine an agent output that would fail the check, the check is not measuring anything useful.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_quality_flywheel"/>

## The Quality Flywheel — Mapped to Sprint Weeks

The evaluation framework is built incrementally. Each phase of the sprint adds a new evaluation layer. Starting at week 1 (not week 9) is essential. Week numbers are sprint-global (weeks 1–12 of the engagement).

**Quality Flywheel Sprint Progression**

<table>
<thead>
<tr>
<th valign="top">Sprint Phase</th>
<th valign="top">Eval Maturity</th>
<th valign="top">What to Add</th>
<th valign="top">Tooling</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

End of Discovery — Spec quality gate

</td>
<td valign="top">

Spec quality (C1–C7)

</td>
<td valign="top">

Run `agentic-eval` against spec and design docs. Focus: C1–C5 spec quality + C7 AI Readiness. Scores below 6/10 on C1–C5 signal Discovery gaps; below "Agent-Ready" on C7 signals spec needs restructuring.

</td>
<td valign="top">

`/agentic-eval` skill

</td>
</tr>
<tr>
<td valign="top">

Weeks 1–3 — Prototype

</td>
<td valign="top">

Traceless

</td>
<td valign="top">

`agent_response` and `rule_compliance` checks on happy-path scenarios. 1 iteration per test case. Measures: `success_rate`, `output_correctness`

</td>
<td valign="top">

`aeval run` locally (traceless)

</td>
</tr>
<tr>
<td valign="top">

Weeks 4–6 — MVP Build

</td>
<td valign="top">

Trace-enabled

</td>
<td valign="top">

Add `tool_call` validations as tool interfaces stabilize. 3 iterations per test case for statistical stability. Measures: + `tool_call_correctness`, `latency`, `total_tokens`

</td>
<td valign="top">

`aeval run` with trace-enabled config; OTel instrumentation required

</td>
</tr>
<tr>
<td valign="top">

Weeks 7–9 — Pre-Deployment

</td>
<td valign="top">

Full suite + code quality gate

</td>
<td valign="top">

Add safety/guardrail checks, boundary conditions, and negative tests. Add `dynamic_conversation` steps for key user journeys. Integrate into CI. Run `agentic-eval` full 11+1-criteria sweep (C6 Spec-to-Code Alignment is the key metric at this point).

</td>
<td valign="top">

`aeval` in GitHub Actions; `/agentic-eval` skill

</td>
</tr>
<tr>
<td valign="top">

Weeks 10–12 — EAC

</td>
<td valign="top">

Production monitoring

</td>
<td valign="top">

Activate online eval — LLM-as-judge scoring against real EAC sessions using the **3-way C/I/N judge** (Complete / Incomplete / Not Applicable). Establish baseline metrics before production go-live. The final two weeks of EAC are the **go-live baseline capture window**: record task completion rate, escalation rate, faithfulness scores, and output embedding centroid as the signed reference baseline for post-deploy drift detection.

</td>
<td valign="top">

`aeval-online` or AI Core CronJob

</td>
</tr>
</tbody>
</table>

**Traceless fallback in offline mode**: when a `response_validations` check would exceed the LLM judge's input-token limit, `aeval` automatically runs the check without trace context and appends a note to the result's `explanations`. The result is still a normal pass/fail — not `skipped`. This fallback applies in offline mode only.

**Synthetic data for early-sprint evaluation:** Before production data is available — typical during weeks 1–6 — the `/fde-ontology` skill (Project LightSpeed Stage 5) can generate domain-contextual synthetic data directly from the logical schema. This gives the agent realistic grounding data to query at runtime and makes evaluation meaningful from week 1, without requiring access to the customer's live systems.

**Week 6 milestone gate**: eval suite runs with ≥ 3 iterations; success rate documented in FDE Gate evidence package.

**Week 9 milestone gate**: full eval suite passes in CI; online eval active in EAC; TR9 offline compliance satisfied.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_success_rate"/>

## The Primary Release Gate: Success Rate

Success rate is binary: 1 if all validations in a test case pass, 0 if any fail. This is the `aeval` framework's primary aggregation metric and the primary release gate metric aligned with [Agent Quality Framework](agent-quality-framework-8204dab.md).

All other metrics are diagnostic:

**Diagnostic Metrics**

<table>
<thead>
<tr>
<th valign="top">Metric</th>
<th valign="top">What It Diagnoses</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

`tool_call_correctness`

</td>
<td valign="top">

Agent calls the right tool with correct parameters

</td>
</tr>
<tr>
<td valign="top">

`output_correctness`

</td>
<td valign="top">

Fraction of `agent_response` validations passing

</td>
</tr>
<tr>
<td valign="top">

`rule_compliance`

</td>
<td valign="top">

Fraction of `rule_compliance` validations passing

</td>
</tr>
<tr>
<td valign="top">

`latency_seconds`

</td>
<td valign="top">

Agent response time; p95 baseline establishes operational norms

</td>
</tr>
<tr>
<td valign="top">

`total_tokens`

</td>
<td valign="top">

Token consumption per invocation; input to cost monitoring (see [Token Efficiency in Industry AI Delivery](token-efficiency-2d8ea45.md))

</td>
</tr>
<tr>
<td valign="top">

`total_cost_usd`

</td>
<td valign="top">

Invocation cost at AI Core pricing; input to floor pricing submission

</td>
</tr>
<tr>
<td valign="top">

`number_of_tool_calls`

</td>
<td valign="top">

Tool call efficiency (see [Agent Quality Framework](agent-quality-framework-8204dab.md), Dimension 1)

</td>
</tr>
</tbody>
</table>

The diagnostic metrics explain *why* success rate is below target, but they are not themselves release gates. An agent with 95% tool call correctness but a low success rate is still failing.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_online_eval"/>

## Online Evaluation in Production

Online evaluation is the continuous quality monitoring layer that runs after go-live. It complements, not replaces, the offline eval suite.

### How It Works

1.  The AI Core CronJob (or `aeval-online` locally) polls MLflow for real user sessions within a configurable lookback window.
2.  Hash-based deterministic sampling ensures consistent coverage across repeated runs.
3.  Each session is scored by an LLM judge using the **3-way C/I/N template** (Complete / Incomplete / Not Applicable) against **requirements** — natural language statements describing expected behavior.
4.  Scores are written back to MLflow as session-level feedback. The MLflow UI shows per-category scores, judge reasoning, and trend analysis over time.

**Not Applicable (N) in online eval**: if a check's criterion was not triggered in a session, the judge returns Not Applicable. Write requirements so their applicability conditions are implicit in the scenario context; avoid encoding "if X does not apply, treat as satisfied" into the check text.

### Requirements Format

```json
[
  {
    "details": "The agent always retrieves the authoritative batch classification result
                before proposing a release quantity",
    "category": "tool_call"
  },
  {
    "details": "The agent expresses all quantities in the canonical unit of measure for
                the material",
    "category": "rule_compliance"
  },
  {
    "details": "When the agent is uncertain about input data quality, it flags this
                uncertainty explicitly rather than proceeding silently",
    "category": "agent_response"
  }
]
```

### Online vs. Offline — Roles and Limits

**Online vs. Offline Eval Comparison**

<table>
<thead>
<tr>
<th valign="top">Dimension</th>
<th valign="top">Offline (aeval run)</th>
<th valign="top">Online (aeval-online)</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Purpose

</td>
<td valign="top">

Assert correctness on known scenarios

</td>
<td valign="top">

Monitor quality trends on real sessions

</td>
</tr>
<tr>
<td valign="top">

When to use

</td>
<td valign="top">

Before every release; in CI

</td>
<td valign="top">

Continuously in production

</td>
</tr>
<tr>
<td valign="top">

Can assert ground truth?

</td>
<td valign="top">

Yes — scripted inputs with known expected outputs

</td>
<td valign="top">

No — designed for trend detection, not ground-truth assertion

</td>
</tr>
<tr>
<td valign="top">

Coverage

</td>
<td valign="top">

Scripted happy paths + boundary conditions

</td>
<td valign="top">

Real user behavior including long tail

</td>
</tr>
</tbody>
</table>

The Supervisor agent (see [Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md)) and online eval are complementary, not redundant. The supervisor reacts in real time within a single agent execution; online eval tracks statistical patterns across sessions over time.

### Relationship to the Post-Deployment Feedback Loop

-   **`aeval` online** provides statistical trend monitoring across sessions. It surfaces regressions but does not prescribe a corrective action.
-   **Feedback loop** is event-driven learning from individual HITL override events. Each override is causally analysed, and the result is either a targeted spec edit, a memory update, or a flag for operator training.

Online eval tells you *something is getting worse*; the feedback loop tells you *what to change* and *whether to change the spec or retrain the operator*.

### Production Drift Types

Not all post-deploy quality degradation is the same. Recognising the drift type before acting matters because it determines the corrective action.

**Production Drift Type Taxonomy**

<table>
<thead>
<tr>
<th valign="top">Drift Type</th>
<th valign="top">How It Manifests</th>
<th valign="top">Key Detection Signal</th>
<th valign="top">Corrective Action</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Data Drift

</td>
<td valign="top">

Users ask different question types; new terminology; language mix changes

</td>
<td valign="top">

PSI > 0.25 on prompt-length or intent distributions; rising embedding distance from baseline centroid

</td>
<td valign="top">

Re-evaluate agent scope; update retrieval corpus

</td>
</tr>
<tr>
<td valign="top">

Concept Drift

</td>
<td valign="top">

Policy/pricing changes; outdated knowledge; seasonal correctness shifts

</td>
<td valign="top">

Declining faithfulness on frozen golden test set; rising escalation rate with stable volume

</td>
<td valign="top">

Update knowledge base; refresh grounding data

</td>
</tr>
<tr>
<td valign="top">

Prompt Regression

</td>
<td valign="top">

Developer modifies system prompt; RAG context injection pattern changes

</td>
<td valign="top">

Abrupt quality score drop correlated with a deployment event; per-prompt-version eval score regression

</td>
<td valign="top">

Rollback prompt to prior version (see [Production Operations](production-operations-42850d6.md)); A/B test prompt changes against baseline

</td>
</tr>
<tr>
<td valign="top">

Model Change Drift

</td>
<td valign="top">

LLM provider silently updates the underlying model

</td>
<td valign="top">

Semantic drift in output embeddings with no correlated prompt change; behaviour diff on frozen canary set across model version boundary

</td>
<td valign="top">

Pin to specific model version; run canary eval on model version boundary before routing production traffic

</td>
</tr>
<tr>
<td valign="top">

Semantic Drift (Distributional)

</td>
<td valign="top">

Responses become increasingly hedged or off-topic

</td>
<td valign="top">

JSD > 0.15–0.20 on output embedding distributions; BERTScore < 0.85 vs. baseline

</td>
<td valign="top">

Root-cause investigation across all other drift types first

</td>
</tr>
<tr>
<td valign="top">

Retrieval Drift (RAG)

</td>
<td valign="top">

Context Precision drops; answers cite stale or irrelevant chunks

</td>
<td valign="top">

RAGAS Context Precision < 0.80; Context Recall < 0.75; rising "no relevant docs" rate

</td>
<td valign="top">

Re-index knowledge base; refresh document corpus; audit embedding model alignment

</td>
</tr>
<tr>
<td valign="top">

Volume Spike Drift

</td>
<td valign="top">

Quality degrades at high query volumes due to infrastructure effects

</td>
<td valign="top">

Quality metric drops correlated with throughput spikes; P99 latency > baseline

</td>
<td valign="top">

Review rate limits; add query queuing; review model serving capacity

</td>
</tr>
</tbody>
</table>

The recommended detection approach for production deployments is a **hybrid sliding + fixed baseline**: a fixed reference baseline captured during the EAC go-live window, combined with a 7-day sliding window for real-time alerts. When any metric crosses its alert threshold, classify the drift type first — this prevents alert fatigue by providing a diagnosis rather than just a signal.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_eval_dimensions"/>

## Evaluation Dimensions Checklist

SAP's AI Golden Path specifies evaluation dimensions with priorities. FDE sprint test suites should cover at minimum the High-priority dimensions before the Week 6 gate, and all Medium-priority dimensions before production go-live.

**Evaluation Dimensions by Priority**

<table>
<thead>
<tr>
<th valign="top">Dimension</th>
<th valign="top">Priority</th>
<th valign="top">Test Case Type</th>
<th valign="top">Notes</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Correctness

</td>
<td valign="top">

High

</td>
<td valign="top">

`agent_response_validations`

</td>
<td valign="top">

Core happy-path checks

</td>
</tr>
<tr>
<td valign="top">

Hallucinations

</td>
<td valign="top">

High

</td>
<td valign="top">

`rule_compliance` checks: "Agent does not state facts not present in retrieved data"

</td>
<td valign="top">

Hard to cover fully; at minimum test known hallucination-prone outputs

</td>
</tr>
<tr>
<td valign="top">

Safety guardrails

</td>
<td valign="top">

High

</td>
<td valign="top">

Boundary tests: inputs designed to trigger guardrails; verify correct Layer 3 escalation

</td>
<td valign="top">

Test that precision guardrails fire correctly on boundary inputs

</td>
</tr>
<tr>
<td valign="top">

Stability

</td>
<td valign="top">

High

</td>
<td valign="top">

Run ≥ 3 iterations per test case; variance in success rate is the stability signal

</td>
<td valign="top">

Low variance = stable; high variance = prompt or model sensitivity problem

</td>
</tr>
<tr>
<td valign="top">

Security

</td>
<td valign="top">

High

</td>
<td valign="top">

Prompt injection tests; tool call boundary tests

</td>
<td valign="top">

Include in MVP eval suite; not just at productization

</td>
</tr>
<tr>
<td valign="top">

Memory

</td>
<td valign="top">

Medium

</td>
<td valign="top">

If agent uses Memory Service: check context retention across turns

</td>
<td valign="top">

</td>
</tr>
<tr>
<td valign="top">

RAG retrieval

</td>
<td valign="top">

Medium

</td>
<td valign="top">

Tool call checks on grounding calls: correct documents retrieved?

</td>
<td valign="top">

</td>
</tr>
<tr>
<td valign="top">

Performance

</td>
<td valign="top">

Medium

</td>
<td valign="top">

`latency_seconds` tracked; p95 baseline established before EAC

</td>
<td valign="top">

</td>
</tr>
<tr>
<td valign="top">

Digression

</td>
<td valign="top">

Low

</td>
<td valign="top">

`rule_compliance`: "Agent does not respond to requests outside its defined scope"

</td>
<td valign="top">

</td>
</tr>
</tbody>
</table>


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_agentic_eval"/>

## Spec Quality Evaluation (agentic-eval)

### What It Evaluates

The `aeval` framework measures whether the agent produces correct *outputs at runtime*. The `agentic-eval` skill measures whether the *spec and codebase* are well-formed: is the spec complete and AI-ready? Does the implementation align with what was specified in Discovery? Are SAP standards and MCP tools being used properly?

These are distinct, complementary dimensions:

**Eval Tool Comparison**

<table>
<thead>
<tr>
<th valign="top">Dimension</th>
<th valign="top">Tool</th>
<th valign="top">What It Measures</th>
<th valign="top">When</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Runtime behavior

</td>
<td valign="top">

`aeval`

</td>
<td valign="top">

Correctness, rule compliance, tool call quality, success rate

</td>
<td valign="top">

Throughout sprint; CI gate

</td>
</tr>
<tr>
<td valign="top">

Spec and build quality

</td>
<td valign="top">

`agentic-eval`

</td>
<td valign="top">

Spec completeness, AI-readiness, code-spec alignment, SAP standards, MCP utilization

</td>
<td valign="top">

End of Discovery; pre-EAC

</td>
</tr>
</tbody>
</table>

### The 11+1 Criteria

**11+1 Agentic-Eval Criteria**

<table>
<thead>
<tr>
<th valign="top">Code</th>
<th valign="top">Criterion</th>
<th valign="top">Weight</th>
<th valign="top">FDE Relevance</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

C1

</td>
<td valign="top">

Spec Completeness

</td>
<td valign="top">

10%

</td>
<td valign="top">

Is the Discovery output implementable without significant clarification?

</td>
</tr>
<tr>
<td valign="top">

C2

</td>
<td valign="top">

Spec Clarity

</td>
<td valign="top">

10%

</td>
<td valign="top">

Could two FDE engineers build essentially the same agent from this spec?

</td>
</tr>
<tr>
<td valign="top">

C3

</td>
<td valign="top">

Testability

</td>
<td valign="top">

10%

</td>
<td valign="top">

Can test cases be derived directly from the spec?

</td>
</tr>
<tr>
<td valign="top">

C4

</td>
<td valign="top">

Traceability Quality

</td>
<td valign="top">

10%

</td>
<td valign="top">

Requirement → code → test chain intact?

</td>
</tr>
<tr>
<td valign="top">

C5

</td>
<td valign="top">

Design Thinking

</td>
<td valign="top">

10%

</td>
<td valign="top">

Are architecture decisions justified and alternatives considered?

</td>
</tr>
<tr>
<td valign="top">

**C6**

</td>
<td valign="top">

**Spec-to-Code Alignment**

</td>
<td valign="top">

**15% (highest)**

</td>
<td valign="top">

Does the built agent match what was specified in Discovery?

</td>
</tr>
<tr>
<td valign="top">

C7

</td>
<td valign="top">

AI Readiness

</td>
<td valign="top">

10%

</td>
<td valign="top">

Is the spec structured for AI implementation? (Prompt-Level / Basic / Agent-Ready / Autonomous-Ready)

</td>
</tr>
<tr>
<td valign="top">

C8

</td>
<td valign="top">

MCP Utilization Effectiveness

</td>
<td valign="top">

10%

</td>
<td valign="top">

Are MCP servers actively used — not just configured? Hard rule: no `.mcp.json` → max 5/10

</td>
</tr>
<tr>
<td valign="top">

C9

</td>
<td valign="top">

SAP Standards Compliance

</td>
<td valign="top">

10%

</td>
<td valign="top">

CAP + Fiori + Hyperspace MCP. Hard rule: no Hyperspace → cannot reach 9–10

</td>
</tr>
<tr>
<td valign="top">

C10

</td>
<td valign="top">

AI-Generated Code Quality

</td>
<td valign="top">

5%

</td>
<td valign="top">

Anti-patterns absent: dead code, hallucinated APIs, over-engineering

</td>
</tr>
<tr>
<td valign="top">

C11

</td>
<td valign="top">

AI Generated vs. Manually Written

</td>
<td valign="top">

classification

</td>
<td valign="top">

Fully AI / AI + Manual / Manual + AI / Fully Manual

</td>
</tr>
<tr>
<td valign="top">

C12

</td>
<td valign="top">

ATQ Skills Adoption

</td>
<td valign="top">

additional

</td>
<td valign="top">

CIS skills installed and actively used in the workflow?

</td>
</tr>
</tbody>
</table>

**C6 carries the highest weight (15%)** because it addresses the most common FDE failure mode: the gap that opens between Discovery specification and Development delivery. Scope creep, late customer feedback, and time pressure cause agents to evolve away from their spec. A low C6 score at pre-EAC means the delivered agent does not match what the customer agreed to in Discovery — a productization risk that must be resolved before handover.

**C7 AI Readiness** is the pre-Development diagnostic. A "Prompt-Level" classification (score 0–2) signals that the spec needs more precision before AI-assisted code generation will be efficient; "Agent-Ready" or "Autonomous-Ready" (6–10) signals the spec will sustain the Development phase without constant clarification loops.

### Two Natural Run Points in the Sprint

**End of Discovery / Develop kickoff — spec quality gate:**
Run against spec and design documents before generating agent code. Focus: C1–C5 (spec quality) and C7 (AI Readiness). A C1–C5 average below 6/10 signals Discovery gaps to resolve before Development; a C7 score below "Agent-Ready" signals the spec needs restructuring. This check takes 5–10 minutes and gives the FDE Delivery Lead a structured, file:line-cited evidence base for the Discovery → Development gate decision.

**Pre-EAC — full quality gate:**
Run against the full project repository before customer go-live. All 11+1 criteria active. C6 (Spec-to-Code Alignment) is the key metric: gaps between specified requirements and implemented capabilities must be resolved or explicitly documented before productization. The HTML or Word report export (`--html`, `--word`) is suitable evidence for the EAC acceptance package and the Sirius QPA test strategy.

### Score Thresholds

**Agentic-Eval Score Thresholds**

<table>
<thead>
<tr>
<th valign="top">Range</th>
<th valign="top">Assessment</th>
<th valign="top">FDE Implication</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

0–2

</td>
<td valign="top">

Critical

</td>
<td valign="top">

Not suitable for delivery; return to Discovery or major remediation

</td>
</tr>
<tr>
<td valign="top">

3–5

</td>
<td valign="top">

Requires improvement

</td>
<td valign="top">

Targeted remediation before EAC go-live

</td>
</tr>
<tr>
<td valign="top">

6–7

</td>
<td valign="top">

Acceptable

</td>
<td valign="top">

Minor improvements; document gaps in production handover package

</td>
</tr>
<tr>
<td valign="top">

8–9

</td>
<td valign="top">

Strong

</td>
<td valign="top">

Ready for EAC with minor improvements

</td>
</tr>
<tr>
<td valign="top">

9–10

</td>
<td valign="top">

Excellent

</td>
<td valign="top">

Production and Agentic Engineering ready

</td>
</tr>
</tbody>
</table>

### Installation

The skill is part of the CIS developer efficiency tools repo (also listed as a Starter Kit component in [Reusable Components and Reference Implementations](reusable-components-1dca6eb.md)). **Contacts / owners: Apurba Ghosh, Saurabh Kejriwal.**

```bash
# Project-level (one engagement)
npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval

# Global (all projects)
npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval -g
```

Invoke as `/agentic-eval [project-path] [--html] [--word]`. The `--html` flag generates a self-contained report with SAP Horizon styling and collapsible per-criterion cards. The `--word` flag generates a Word document in the Porsche sample report style. Both formats are suitable as FDE Gate or EAC acceptance evidence artifacts.


<a name="loiof940c93e5f5a46a99a5dc358e66d4325__section_fde_delivery"/>

## FDE Delivery Implications

### During Discovery

-   For each numerical output or high-risk agent action in the TO-BE process flow, define at least one boundary condition test case: what input should trigger the guardrail? What input should pass? Document these in the Draft Product Design Specs.
-   Confirm which SAP calculation services exist in the customer's landscape — they determine whether `tool_call` validations against authoritative references are feasible.
-   Agree with the process owner what "success" means for each user journey. This agreement becomes the `agent_response_validations` in the test case.
-   Run the G1–G4 readiness gates against the compiled discovery outputs before any agent spec is generated. Gate scores below 0.8 should be resolved — or their coaching questions answered — before the team enters the Development phase.
-   Run `agentic-eval` against the Discovery output documents at the end of Discovery. A C1–C5 average below 6/10 or a C7 score below "Agent-Ready" should be resolved before Develop kickoff.

### During Develop

-   Week 1: create the first test cases (traceless). Even three `agent_response` checks on the happy-path user journey are enough to establish the evaluation baseline. If production data is not yet available for agent grounding, generate synthetic data using the `/fde-ontology` skill (Project LightSpeed Stage 5).
-   Keep test cases in version control alongside the agent workflow. Treat a failing test case the same way a failing unit test is treated: do not proceed to the next sprint phase with known failing checks.
-   The eval report (HTML format from `aeval`) is the evidence package for the FDE Gate. Produce it before the Week 6 review, not during it.
-   Run the post-build scorecard before the Week 9 deployment gate. A generation drift failure (Category B) at Week 9 is recoverable but disruptive; catching it earlier during iterative development avoids last-minute blockers.
-   Run `agentic-eval` against the full project repo in the Weeks 7–9 window — before EAC, not during it. C6 (Spec-to-Code Alignment) is the metric to watch.
-   A2A endpoint exposure (TR1) should be implemented early — the eval harness invokes agents via A2A. An agent without an A2A endpoint cannot be evaluated by `aeval` and therefore cannot satisfy TR9.

### During Productization

-   The offline eval suite must be running in CI (TR9) before Sirius productization tasks begin. The QPA test strategy references eval coverage.
-   The online eval configuration (requirements JSON, lookback window, sampling rate) must be part of the production handover package.
-   Configure the post-deployment feedback loop outcome horizons to match the business process: financial postings may require a 24-hour confirmation window; supply chain decisions may require longer. Outcomes that return `outcome_confirmed_bad` escalate to CRITICAL and require human review before any spec or memory update is applied.
-   Re-run the full eval suite against any model version change (TR9 implication). A model update that degrades success rate by more than 5 percentage points from baseline is a rollback trigger (see [Production Operations](production-operations-42850d6.md)).

**Related Information**  

[Key Capabilities — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/3f7ceafb1eb24d00ac50bf3499f50688.html) :arrow_upper_right:

[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html) :arrow_upper_right:

[Joule Studio Glossary](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/1e214183c85c4e4faff59a96c724f4fb.html) :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html) :arrow_upper_right:
