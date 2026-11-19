[//]: # (loio: 8cabb2d)

# Chapter 11: Agent Evaluation in FDE Sprints

## 11.1 Why Evaluation Needs Its Own Chapter

Chapter 1 defines *what* to measure: three quality dimensions (technical, process KPIs,
business outcomes) and the success rate as the primary release gate. This chapter defines
*how* to measure it — the mechanism, the tooling, and when in the sprint each piece is
required.

The distinction matters because evaluation in an FDE context has properties that make
informal approaches fail:

- **Agents are non-deterministic.** The same input can produce different outputs on
  different runs. A single manual test run is not quality evidence; a structured test
  suite run across multiple iterations is.
- **TR9 is mandatory.** SAP's AI Golden Path requires both offline evaluation (scripted
  test suites in CI) and online evaluation (monitoring of live production sessions) for
  every production agent. This is not a recommendation.
- **Evaluation must start early.** FDE sprints are short (target: 90 days). An evaluation
  infrastructure built in week 9 cannot generate the quality evidence needed for the FDE
  Gate at week 6 or the EAC launch at week 9.

---

## 11.2 Three-Phase Evaluation Lifecycle

Evaluation in FDE sprints is not a single activity — it spans three distinct phases, each
with its own tools, owners, and purpose. Understanding the lifecycle prevents the most
common failure mode: teams that treat evaluation as a final-sprint activity and find they
lack the quality evidence needed for the FDE Gate.

| Phase | When | Tool | Purpose |
|---|---|---|---|
| Pre-build | Before agent spec is generated | G1–G4 Readiness Gates | Verify discovery outputs are complete enough to build from |
| Spec quality | End of Discovery; pre-EAC | `agentic-eval` skill (11+1 criteria) | Verify spec is complete and AI-ready; verify code-spec alignment before customer go-live |
| Post-build | After agent code is generated, before deploy | 6-category eval scorecard | Verify agent spec, Joule config, routing quality, and compliance with MUST rules |
| Post-deployment | Continuously in production | Two-path feedback loop + aeval online eval | Detect regressions, close learning loops, improve agent over time |

The `aeval` offline/online framework (§11.7–§11.11) covers the CI-gated release gate between
post-build and post-deployment. The G1–G4 gates (§11.3) and the 6-category scorecard (§11.4)
are additions from the Project LightSpeed delivery toolchain that extend evaluation earlier —
into the discovery and code-generation phases — where defects are cheapest to fix. The
`agentic-eval` skill (§11.13) adds a fourth dimension: structured spec-and-build quality
measurement at the two points in the sprint where spec drift is most likely to surface.

---

## 11.3 Pre-Build Readiness Gates (G1–G4)

Before generating an agent spec, the extracted knowledge must meet four readiness gates.
These gates prevent the most common cause of wasted build effort: generating agent code from
discovery outputs that are structurally incomplete. Each gate scores 0–1:

- **≥ 0.8** — proceed
- **≥ 0.5 and < 0.8** — proceed with gaps; coaching questions auto-generated
- **< 0.5** — stop and return to discovery

| Gate | What it checks | Pass threshold |
|---|---|---|
| G1 — Entity Completeness | Proportion of entities successfully resolved against SAP's EKX/canonical catalog | ≥ 0.8 |
| G2 — Goal Coverage | Every extracted Capability has a parent Goal | ≥ 0.8 |
| G3 — Actor-Process Binding | Every extracted Process has an assigned Actor | ≥ 0.8 |
| G4 — Integration Readiness | Every Integration has an API binding | ≥ 0.8 |

When a gate score falls between 0.5 and 0.8, the system auto-generates **coaching questions** —
targeted gap-filling queries designed to be taken back to the customer in the next working session.
This transforms discovery gaps from silent blockers into structured follow-up items that can be
tracked and closed.

The 0.5 hard stop is deliberate: generating an agent spec from a foundation that is less than
half-resolved against SAP's canonical catalog produces agent code that cannot correctly route to
the right APIs, cannot express entities in terms the system of record understands, and will fail
the post-build scoring categories A, C, and F before any functionality is tested.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-synthesize` skill.

---

## 11.4 Post-Build Evaluation Scorecard

After agent code is generated and before deployment, a 6-category scorecard validates
readiness. This is a hybrid evaluation: categories A–C are deterministic checks that
execute in milliseconds; categories D–F are LLM-graded. Output: `evals/eval-report.md`.

| Category | What it checks | Method |
|---|---|---|
| A — Spec integrity | `agent.spec.json` is valid, complete, and internally consistent | Deterministic |
| B — Generation drift | `spec_sha256` in generated code matches the source spec | Deterministic |
| C — Joule config | Joule capability bundle structure, naming invariants, schema validation via Joule MCP | Deterministic |
| D — Routing description quality | Each agent's routing description is distinctive, domain-specific, contains no personal pronouns, and does not overlap with other agents | LLM-graded |
| E — MUST-rule compliance | Business constraints encoded as MUST rules (from `golden_scenarios.yaml`) are satisfied | LLM-graded |
| F — Coverage / completeness | All required capabilities have corresponding agent behaviors | LLM-graded |

**Category B — generation drift** is the most operationally critical check. If
`agent.spec.json` is edited after code generation — common in iterative FDE development,
where late-breaking customer feedback prompts spec adjustments — the generated agent no
longer implements the spec it claims to implement. A `spec_sha256` mismatch is a blocking
failure: the deployment is invalid regardless of whether the other categories pass. The
fix is to re-run code generation from the updated spec, then re-evaluate.

**Category D — routing description quality** addresses a failure mode that is invisible
until production: two agents with overlapping or generic routing descriptions cause the
Joule orchestrator to misroute user requests. Deterministic testing cannot catch this
because it is a semantic distinctness problem, not a schema problem. The LLM-graded check
compares all routing descriptions in the bundle against each other and flags overlap risk
before deployment.

**MUST-rule compliance (Category E)** connects to Chapter 4 (Precision Guardrails): the
MUST rules encoded in `golden_scenarios.yaml` are the machine-verifiable form of the
business constraints that the guardrail design was built to enforce. Passing Category E
is the pre-deploy signal that the agent's behavior is consistent with those constraints
under the test scenarios.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-evals` skill.

---

## 11.5 Post-Deployment Feedback Loop

The feedback loop captures every HITL override event and transforms it into a learning
signal. It addresses the gap between online eval (which detects *that* something is wrong
in aggregate) and knowing *what* to change and *whether* a human override represents a
genuine agent error.

The loop runs on two parallel paths:

**Spec path (human-gated):** HITL override → analyst LLM proposes targeted edit to
`agent.spec.json` → human approves → redeploy triggered. This path fires only when the
spec genuinely needs to change. A human gate is mandatory because spec changes affect all
future deployments.

**Memory path (autonomous):** Per-conversation traces → analyst LLM proposes memory
changes → auditor LLM confirms or rejects → ACTIVE memories are injected into the agent's
system prompt on subsequent turns. No human gate for memory updates; the auditor LLM
provides the quality check. This path handles corrections that are context-specific and
do not require a spec change.

Both paths handle the same four learning cases:

| Case | What happened | Action |
|---|---|---|
| Case 1 — Agent wrong + human overrode correctly | Agent made an error; human catch was correct | Spec path: propose targeted spec fix. Memory path: add corrective memory. |
| Case 2 — Both wrong | Agent made an error; human override was also incorrect | No spec change. Flag for human review. Do NOT propagate the override. |
| Case 3 — Agent correct + human overrode incorrectly | Agent was right; human override introduced an error | **Critical: do NOT propose a spec change.** Flag the override as an operator training issue. Track separately. |
| Case 4 — Agent correct + human agreed, outcome confirmed | Agent performed correctly; outcome confirmed good | Reinforce via memory path. No spec change needed. |

**Case 3 — the operator training issue** is the most subtle and the most consequential.
Without explicit handling of this case, a feedback system learns incorrect behaviors from
operator mistakes. The feedback loop is not a majority-vote system: it is a causal analysis
system. Every implementation of HITL feedback must handle Case 3 explicitly, with a
separate tracking path that feeds into operator training, not agent spec revision.

The Case 3 tracking path works as follows: when the analyst LLM determines Case 3 (agent
correct, human wrong), it logs the override event with a `case_3` flag and the specific
agent output and human decision. These flagged events are surfaced to the engagement's
Adoption Lead through the **feedback loop's own reporting surface** as a distinct metric —
**"agent correct, human override rate"** — separate from the general HITL override rate.
(This metric belongs to the feedback loop, not the aeval online-eval trend dashboard; the
two systems are deliberately kept separate — see §11.11.) A rising
`agent correct, human override rate` is an adoption signal, not a quality signal; it should
trigger an operator training review (Chapter 12 §12.7), not a prompt revision. The
distinction matters: acting on Case 3 events as if they were Case 1 events degrades the
agent by teaching it to match human mistakes. See Chapter 12 §12.7 for the HITL override
guide that defines what constitutes a legitimate override vs. an operator error requiring
training intervention.

**Deferred outcome tracking:** Not all outcomes are immediately observable. A financial
posting might take 24 hours to be confirmed correct; a batch release might generate
quality results only after manufacturing completes. The feedback loop supports configurable
outcome horizons: the system waits for a downstream outcome event before closing the
learning loop. Outcomes that come back as `outcome_confirmed_bad` escalate the associated
memory or spec change to CRITICAL severity, requiring human review before any update is
applied.

Implementation reference: `context/fde/project-lightspeed/` — `/fde-feedback` skill.

---

## 11.6 Testing vs. Evaluation — Two Distinct Activities

SAP's AI Golden Path draws a deliberate distinction that FDE teams should internalize:

| Activity | Purpose | When | Owner |
|---|---|---|---|
| **Testing** | Ongoing defect finding throughout development | Continuously, throughout Develop (formerly Execute) | FDE Application Engineers |
| **Evaluation** | Milestone-based comparison: has the agent improved between versions? | At sprint milestones (Week 6 MVP, Week 9 pre-deployment, each productization release) | FDE Dev Architect + Domain PM |

Both are required. Testing uses unit tests, integration tests, and smoke tests. Evaluation
uses structured test suites that measure agent behavior on representative user journeys
across multiple iterations.

The distinction also maps to tools: testing uses standard test frameworks (pytest, jest);
evaluation uses SAP's `aeval` framework, which is the subject of this chapter.

---

## 11.7 The aeval Framework

### What It Is

`aeval` is SAP's internal Python framework and CLI for evaluating AI agents. It implements
the two evaluation modes required by TR9:

- **Offline mode** (`aeval run`): scripted test cases run against a live or local agent.
  Results exported as JSON and HTML reports. Gates release decisions.
- **Online mode** (`aeval-online` / AI Core CronJob): real production traces retrieved from
  MLflow and scored by an LLM judge. Results written back to MLflow as session-level
  feedback. Tracks production quality trends continuously.

Both modes use the same evaluation core: SAP's `agent-inspect` library, which invokes
3 independent LLM judge calls per validation (majority vote) via SAP AI Core GenAI Hub.
The 3-judge configuration was determined empirically — it achieves reliability close to
that of a 5-judge ensemble while materially reducing latency and cost.

### Provisioning aeval on FDE Engagements

`aeval` is available from the `developer-efficiency-tools` repository
(`github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools`).
Installation on a new engagement environment:

```bash
npx skills add aeval-set-up  # installs the setup skill
# Then follow the /aeval-set-up instructions to configure:
# - GenAI Hub connection (required for LLM judge calls)
# - MLflow instance (required for online eval trace storage)
# - AI Core Extended plan (required for aeval-online CronJob)
```

**Configuration requirements:**
- **GenAI Hub connection**: the LLM judge calls require a GenAI Hub deployment accessible
  from the eval environment. For FDE sprint environments, use the team's shared AI Core
  development instance. For production online eval, the `aeval-online` CronJob must have
  access to a GenAI Hub deployment in the same tenant as the agent.
- **MLflow setup**: offline eval results are stored in MLflow. The LightSpeed-provided
  MLflow instance (`references/observe-and-eval/`) is the standard starting point; for
  production online eval, provision a dedicated MLflow server in the customer's BTP
  sub-account.
- **Sovereign regions**: `aeval-online` requires a GenAI Hub endpoint within the sovereign
  boundary for judge calls. In CN40, NS2, and KSA, confirm that a GenAI Hub deployment
  is available in the sovereign region before committing to online eval in production.
  If not available, online eval must be deferred or run via a local judge model — discuss
  the alternative with the BAIP platform team during Discovery.

### Tracing Modes

The eval framework's depth depends on what tracing data is available:

| Tracing mode | What it requires | Metrics available |
|---|---|---|
| **Traceless** | Nothing — no OTel setup needed | `output_correctness`, `rule_compliance`, `success_rate` |
| **Trace-enabled** | OTel-compliant traces from BAF or MLflow | All traceless metrics + `tool_call_correctness`, `latency_seconds`, `total_tokens`, `total_cost_usd`, `number_of_tool_calls` |
| **Online eval** | Production traces in MLflow | Per-session `success_rate`, `token_count`, `latency` — continuous monitoring |

**Traceless mode is deliberately available for early-sprint use.** An agent without
full OTel instrumentation can still be evaluated for output correctness and rule compliance
from sprint week 1. This means evaluation starts before instrumentation is complete.

### OTel Score Schema (November 2026)

The aeval OTel output uses `gen_ai.evaluation.*` attributes. Three behavioral rules that
matter for anyone consuming eval traces or building dashboards on top of them:

| Score type | OTel behavior |
|---|---|
| **Numeric scores** (float, e.g., 0.85) | Written to `gen_ai.evaluation.score.value` (double) |
| **Label/categorical scores** (string, e.g., "high") | Written to `gen_ai.evaluation.score.label` — **not** to `score.value`. Do not attempt to read categorical results from `score.value`. |
| **Boolean scores** | Both `score.value: 1.0\|0.0` AND `score.label: "true"\|"false"` emitted simultaneously |
| **NOT_APPLICABLE scores** | **Not emitted at all.** Absence in the trace is the signal — a check that did not apply to a session produces no OTel event. Do not equate "no record" with "score = 0". |
| **Error-skipped scores** | `error.type` attribute set to one of: `JUDGE_FAILED`, `AGGREGATION_FAILED`, `TRACE_UNAVAILABLE`, `UNMAPPED_LABEL`. `score.value` and `score.label` are omitted. |

**Dashboard implication**: aggregate success-rate calculations must explicitly handle absent records (NOT_APPLICABLE) vs. zero-scored records (Incomplete). A query that treats missing rows as 0 will silently under-report success rates for checks with frequent NA cases.

---

## 11.8 The Test Case: the Primary Evaluation Artifact

For each FDE sprint agent, the test case file is a sprint deliverable at the same level
as the Joule Studio workflow. It should be version-controlled alongside the agent and
reviewed at sprint milestones.

### Test Case DSL

Test cases are YAML files. Each test case has one or more steps; each step has an
input message and a set of validations:

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

  - type: fixed_message
    input_message: "The standard batch size is 1000 kg. What quantity do you recommend?"
    agent_response_validations:
      - check: "Agent recommends a quantity consistent with the 1000 kg standard and any
          applicable batch classification tolerance"
      - check: "Agent expresses the quantity in kg (not metric tons or other units)"
```

### Two Step Types

| Step type | When to use | What it tests |
|---|---|---|
| `fixed_message` | Scripted happy-path and boundary tests; known inputs | Output correctness, tool call compliance, rule adherence on deterministic inputs |
| `dynamic_conversation` | Multi-turn workflows where user wording is variable | Agent's ability to guide a conversation to a goal across natural language variation |

For dynamic conversations, a **User Agent** (LLM-powered) drives the conversation from
a `task_summary`. The agent under test must reach the defined termination conditions
within `max_turns` (maximum: 15):

```yaml
  - type: dynamic_conversation
    user_agent:
      task_summary: "Guide the agent to release a batch with quantity confirmed by the
          batch classification service. You are the process owner."
      termination_conditions:
        - check: "Agent has confirmed the release quantity and requested approval"
      initial_message: "I need to process batch release for production order 500012345"
    max_turns: 8
    agent_response_validations:
      - check: "Agent arrives at a specific quantity recommendation backed by the
          batch classification service"
    tool_validations:
      expected_tool_calls:
        - tool: sapTaskCenter_submit
          output:
            check: "Approval task created with proposed quantity and reference value"
```

### What Makes a Good Check

The quality of the evaluation depends on the quality of the checks. Vague checks
produce misleading eval results; specific checks produce actionable ones:

| Check quality | Example |
|---|---|
| Vague — avoid | "Agent responds helpfully about the batch" |
| Specific — preferred | "Agent states a quantity in kg within the range 800–1200 for this production order" |
| Tool-level | "Agent calls `get_batch_classification_result` with `order_id = '500012345'`" |
| Safety/guardrail | "Agent does not commit a quantity to the system of record without triggering the Layer 3 approval check" |

The rule of thumb: a check should be falsifiable. If it is impossible to imagine an
agent output that would fail the check, the check is not measuring anything useful.

---

## 11.9 The Quality Flywheel — Mapped to Sprint Weeks

The evaluation framework is built incrementally. Each phase of the sprint adds a new
evaluation layer. Starting at week 1 (not week 9) is essential. Week numbers throughout
this table are **sprint-global (weeks 1–12 of the engagement)**:

| Sprint phase | Eval maturity | What to add | Tooling |
|---|---|---|---|
| End of Discovery — Spec quality gate | Spec quality (C1–C7) | Run `agentic-eval` against spec and design docs. Focus: C1–C5 spec quality + C7 AI Readiness. Scores below 6/10 on C1–C5 signal Discovery gaps to resolve before Develop; below "Agent-Ready" on C7 signals spec needs restructuring before agentic code generation is efficient | `/agentic-eval` skill |
| Weeks 1–3 — Prototype | Traceless | `agent_response` and `rule_compliance` checks on happy-path scenarios. 1 iteration per test case. Measures: `success_rate`, `output_correctness` | `aeval run` locally (traceless) |
| Weeks 4–6 — MVP build | Trace-enabled | Add `tool_call` validations as tool interfaces stabilize. 3 iterations per test case for statistical stability. Measures: + `tool_call_correctness`, `latency`, `total_tokens` | `aeval run` with trace-enabled config; OTel instrumentation required |
| Weeks 7–9 — Pre-deployment | Full suite + code quality gate | Add safety/guardrail checks, boundary conditions, and negative tests. Add `dynamic_conversation` steps for key user journeys. Integrate into CI. Run `agentic-eval` full 11+1-criteria sweep (C6 Spec-to-Code Alignment is the key metric at this point — gaps represent productization risk) | `aeval` in GitHub Actions; `/agentic-eval` skill |
| Weeks 10–12 — EAC | Production monitoring | Activate online eval — LLM-as-judge scoring against real EAC sessions using the **3-way C/I/N judge** (Complete / Incomplete / Not Applicable). Define production monitoring requirements (natural language statements). Establish baseline metrics before production go-live. The final two weeks of EAC are the **go-live baseline capture window**: record task completion rate, escalation rate, faithfulness scores, and output embedding centroid as the signed reference baseline for post-deploy drift detection (§11.11). | `aeval-online` or AI Core CronJob |

**Traceless fallback in offline mode**: when a `response_validations` check would exceed the LLM judge's input-token limit (due to large traces), aeval automatically runs the check without trace context and appends a note to the result's `explanations`. The result is still a normal pass/fail — not `skipped`. This fallback applies in offline mode only; in online eval, trace data is the input so no fallback exists.

**Synthetic data for early-sprint evaluation:** Before production data is available —
typical during weeks 1–6 — the `/fde-ontology` skill (Project LightSpeed Stage 5) can
generate domain-contextual synthetic data directly from the logical schema. Using Claude
to infer realistic value ranges per entity (respecting FK relationships and SAP field naming
conventions), it produces a `mock_data.db` SQLite snapshot that can be pushed to HANA Cloud
KG via `/fde-materializekg`. This gives the agent realistic grounding data to query at
runtime and makes evaluation meaningful from week 1, without requiring access to the
customer's live systems. The same synthetic data can be replaced with live system data
in later sprint phases without changing the eval suite structure.

**Week 6 milestone gate**: eval suite runs with ≥ 3 iterations; success rate documented
in FDE Gate evidence package. This is the technical grounding for the feasibility
assessment by Product Engineering.

**Week 9 milestone gate**: full eval suite passes in CI; online eval active in EAC;
TR9 offline compliance satisfied.

---

## 11.10 The Primary Release Gate: Success Rate

Success rate is binary: 1 if all validations in a test case pass, 0 if any fail. This
is the aeval framework's primary aggregation metric and the primary release gate metric
aligned with Chapter 1 section 1.2.

All other metrics are diagnostic:

| Metric | What it diagnoses |
|---|---|
| `tool_call_correctness` | Agent calls the right tool with correct parameters |
| `output_correctness` | Fraction of `agent_response` validations passing |
| `rule_compliance` | Fraction of `rule_compliance` validations passing |
| `latency_seconds` | Agent response time; p95 baseline establishes operational norms |
| `total_tokens` | Token consumption per invocation; input to cost monitoring (Chapter 10) |
| `total_cost_usd` | Invocation cost at AI Core pricing; input to floor pricing submission |
| `number_of_tool_calls` | Tool call efficiency (Chapter 1 §1.2 Dimension 1) |

The diagnostic metrics explain *why* success rate is below target, but they are not
themselves release gates. An agent with 95% tool call correctness but a low success rate
is still failing — the failing validation is where the investigation starts.

---

## 11.11 Online Evaluation in Production

Online evaluation is the continuous quality monitoring layer that runs after go-live.
It complements, not replaces, the offline eval suite.

### How It Works

1. The AI Core CronJob (or `aeval-online` locally) polls MLflow for real user sessions
   within a configurable lookback window.
2. Hash-based deterministic sampling ensures consistent coverage across repeated runs.
3. Each session is scored by an LLM judge using the **3-way C/I/N template**
   (Complete / Incomplete / Not Applicable) against **requirements** — natural language
   statements describing expected behavior.
4. Scores are written back to MLflow as session-level feedback. The MLflow UI shows
   per-category scores, judge reasoning, and trend analysis over time.

**Not Applicable (N) in online eval**: if a check's criterion was not triggered in a
session (e.g., a refund eligibility check on a session where no refund was requested),
the judge returns Not Applicable. This produces no OTel event — see §11.7 OTel schema.
Write requirements so their applicability conditions are implicit in the scenario context;
avoid encoding "if X does not apply, treat as satisfied" into the check text — this
confuses the judge between N and C and degrades scoring reliability.

### Requirements Format

Requirements for online eval are natural language statements in three categories:

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

| Dimension | Offline (aeval run) | Online (aeval-online) |
|---|---|---|
| Purpose | Assert correctness on known scenarios | Monitor quality trends on real sessions |
| When to use | Before every release; in CI | Continuously in production |
| Can assert ground truth? | Yes — scripted inputs with known expected outputs | No — designed for trend detection, not ground-truth assertion |
| Regression detection | Yes — compare success rate across versions | Yes — moving average trends surface regressions |
| Coverage | Scripted happy paths + boundary conditions | Real user behavior including long tail |

**The Supervisor agent (Chapter 2) and online eval are complementary, not redundant.**
The supervisor reacts in real time within a single agent execution; online eval tracks
statistical patterns across sessions over time. A supervisor catching 3 anomalies per
week may not surface a trend; online eval scoring 200 sessions per week will.

### Relationship to the Post-Deployment Feedback Loop

The aeval online evaluation (§11.11) and the post-deployment feedback loop (§11.5) are
complementary, not redundant:

- **aeval online** provides statistical trend monitoring across sessions. It surfaces
  regressions — a drop in session-level success rate or rule compliance over time — but
  does not prescribe a corrective action.
- **Feedback loop** is event-driven learning from individual HITL override events. Each
  override is causally analysed (the four learning cases in §11.5), and the result is
  either a targeted spec edit, a memory update, or a flag for operator training. It closes
  specific gaps rather than monitoring aggregate trends.

In practice: online eval tells you *something is getting worse*; the feedback loop tells
you *what to change* and *whether to change the spec or retrain the operator*.

### Production Drift Types

Not all post-deploy quality degradation is the same. Recognising the drift type before
acting matters because it determines the corrective action. The following taxonomy covers
the principal drift types relevant to SAP agent deployments:

| Drift Type | How It Manifests | Key Detection Signal | Corrective Action |
|---|---|---|---|
| Data Drift | Users ask different question types; new terminology; language mix changes | PSI > 0.25 on prompt-length or intent distributions; rising embedding distance from baseline centroid | Re-evaluate agent scope; update retrieval corpus |
| Concept Drift | Policy/pricing changes; outdated knowledge; seasonal correctness shifts | Declining faithfulness on frozen golden test set; rising escalation rate with stable volume | Update knowledge base; refresh grounding data |
| Prompt Regression | Developer modifies system prompt; RAG context injection pattern changes | Abrupt quality score drop correlated with a deployment event; per-prompt-version eval score regression | Rollback prompt to prior version (§8.3); A/B test prompt changes against baseline |
| Model Change Drift | LLM provider silently updates the underlying model | Semantic drift in output embeddings with no correlated prompt change; behaviour diff on frozen canary set across model version boundary | Pin to specific model version (§8.3); run canary eval on model version boundary before routing production traffic |
| Semantic Drift (Distributional) | Responses become increasingly hedged or off-topic; aggregate semantic character shifts | JSD > 0.15–0.20 on output embedding distributions; BERTScore < 0.85 vs. baseline | Root-cause investigation across all other drift types first |
| Retrieval Drift (RAG) | Context Precision drops; answers cite stale or irrelevant chunks | RAGAS Context Precision < 0.80; Context Recall < 0.75; rising "no relevant docs" rate | Re-index knowledge base; refresh document corpus; audit embedding model alignment |
| Volume Spike Drift | Quality degrades at high query volumes due to infrastructure effects | Quality metric drops correlated with throughput spikes; P99 latency > baseline | Review rate limits; add query queuing; review model serving capacity |

The recommended detection approach for production deployments is a **hybrid sliding + fixed
baseline**: a fixed reference baseline captured during the EAC go-live window, combined with a
7-day sliding window for real-time alerts. When any metric crosses its alert threshold, classify
the drift type first — this prevents alert fatigue by providing a diagnosis rather than just a
signal, and routes the investigation to the correct corrective action.

**See also:** The *AI Quality Drift Monitor for Customer Deployments*
(`documents/AI_Quality_Drift_Monitor_FDE.pdf`) is a dedicated discovery concept specifying how
to operationalize post-deploy drift detection for FDE customer deployments. It defines the
go-live baseline capture procedure, a three-tier metrics framework (Tier 1 operational / Tier 2
LLM-judge semantic / Tier 3 distributional statistics), drift classification and alerting pipeline
(P0–P3 severity tiers), and a regression test feedback loop that automatically promotes confirmed
drift events to new golden test cases in the evaluation suite. The recommended implementation
builds on top of `aeval`/MLflow.

---

## 11.12 Evaluation Dimensions Checklist

SAP's AI Golden Path specifies evaluation dimensions with priorities. FDE sprint test
suites should cover at minimum the High-priority dimensions before the Week 6 gate,
and all Medium-priority dimensions before production go-live:

| Dimension | Priority | Test case type | Notes |
|---|---|---|---|
| Correctness | High | `agent_response_validations` | Core happy-path checks |
| Hallucinations | High | `rule_compliance` checks: "Agent does not state facts not present in retrieved data" | Hard to cover fully; at minimum test known hallucination-prone outputs |
| Safety guardrails | High | Boundary tests: inputs designed to trigger guardrails (Chapter 4); verify correct Layer 3 escalation | Test that precision guardrails fire correctly on boundary inputs |
| Stability | High | Run ≥ 3 iterations per test case; variance in success rate is the stability signal | Low variance = stable; high variance = prompt or model sensitivity problem |
| Summarization | High | `agent_response` checks on outputs involving summarization of retrieved data | |
| Security | High | Prompt injection tests (Chapter 3 §3.7); tool call boundary tests | Include in MVP eval suite; not just at productization |
| Memory | Medium | If agent uses Memory Service: check context retention across turns | |
| RAG retrieval | Medium | Tool call checks on grounding calls: correct documents retrieved? | |
| Performance | Medium | `latency_seconds` tracked; p95 baseline established before EAC | |
| UI/UX | Medium | `agent_response` checks: tone, structure, brevity of responses | |
| Digression | Low | `rule_compliance`: "Agent does not respond to requests outside its defined scope" | |

---

## 11.13 Spec Quality Evaluation (agentic-eval)

### What It Evaluates

The `aeval` framework (§11.7) measures whether the agent produces correct *outputs at runtime*.
The `agentic-eval` skill measures whether the *spec and codebase* are well-formed: is the spec
complete and AI-ready? Does the implementation align with what was specified in Discovery?
Are SAP standards and MCP tools being used properly?

These are distinct, complementary dimensions:

| Dimension | Tool | What it measures | When |
|---|---|---|---|
| Runtime behavior | `aeval` (§11.7–§11.11) | Correctness, rule compliance, tool call quality, success rate | Throughout sprint; CI gate |
| Spec & build quality | `agentic-eval` | Spec completeness, AI-readiness, code-spec alignment, SAP standards, MCP utilization | End of Discovery; pre-EAC |

Both dimensions can fail independently: an agent can pass all runtime eval checks while
still drifting significantly from its original spec (C6 gap), or it can have a complete spec
but produce outputs that fail correctness checks. FDE teams need both signals.

### The 11+1 Criteria

| Code | Criterion | Weight | FDE Relevance |
|------|-----------|--------|---------------|
| C1 | Spec Completeness | 10% | Is the Discovery output implementable without significant clarification? |
| C2 | Spec Clarity | 10% | Could two FDE engineers build essentially the same agent from this spec? |
| C3 | Testability | 10% | Can test cases be derived directly from the spec? |
| C4 | Traceability Quality | 10% | Requirement → code → test chain intact? |
| C5 | Design Thinking | 10% | Are architecture decisions justified and alternatives considered? |
| **C6** | **Spec-to-Code Alignment** | **15% (highest)** | Does the built agent match what was specified in Discovery? |
| C7 | AI Readiness | 10% | Is the spec structured for AI implementation? (Prompt-Level / Basic / Agent-Ready / Autonomous-Ready) |
| C8 | MCP Utilization Effectiveness | 10% | Are MCP servers actively used — not just configured? Hard rule: no `.mcp.json` → max 5/10 |
| C9 | SAP Standards Compliance | 10% | CAP + Fiori + Hyperspace MCP. Hard rule: no Hyperspace → cannot reach 9–10 |
| C10 | AI-Generated Code Quality | 5% | Anti-patterns absent: dead code, hallucinated APIs, over-engineering |
| C11 | AI Generated vs. Manually Written | classification | Fully AI / AI + Manual / Manual + AI / Fully Manual |
| C12 | ATQ Skills Adoption | additional | CIS skills installed and actively used in the workflow? |

**C6 carries the highest weight (15%)** because it addresses the most common FDE failure mode:
the gap that opens between Discovery specification and Develop delivery. Scope creep, late customer
feedback, and time pressure cause agents to evolve away from their spec. A low C6 score at
pre-EAC means the delivered agent does not match what the customer agreed to in Discovery —
a productization risk that must be resolved before handover.

**C7 AI Readiness** is the pre-Development diagnostic. A "Prompt-Level" classification (score 0–2)
signals that the spec needs more precision before AI-assisted code generation will be efficient;
"Agent-Ready" or "Autonomous-Ready" (6–10) signals the spec will sustain the Development phase
without constant clarification loops.

### Two Natural Run Points in the Sprint

**End of Discovery / Develop kickoff — spec quality gate:**
Run against spec and design documents before generating agent code. Focus: C1–C5
(spec quality) and C7 (AI Readiness). A C1–C5 average below 6/10 signals Discovery gaps
to resolve before Develop; a C7 score below "Agent-Ready" signals the spec needs restructuring
before agentic development will be efficient. This check takes 5–10 minutes and gives the
FDE Delivery Lead a structured, file:line-cited evidence base for the Discovery → Develop
gate decision.

**Pre-EAC — full quality gate:**
Run against the full project repository before customer go-live. All 11+1 criteria active.
C6 (Spec-to-Code Alignment) is the key metric: gaps between specified requirements and
implemented capabilities must be resolved or explicitly documented before productization.
C9 (SAP Standards Compliance) confirms CAP/Fiori adherence and Hyperspace MCP integration.
The HTML or Word report export (`--html`, `--word`) is suitable evidence for the EAC
acceptance package and the Sirius QPA test strategy.

### Score Thresholds

| Range | Assessment | FDE implication |
|-------|-----------|-----------------| 
| 0–2 | Critical | Not suitable for delivery; return to Discovery or major remediation |
| 3–5 | Requires improvement | Targeted remediation before EAC go-live |
| 6–7 | Acceptable | Minor improvements; document gaps in production handover package |
| 8–9 | Strong | Ready for EAC with minor improvements |
| 9–10 | Excellent | Production and Agentic Engineering ready |

### Relationship to the Post-Build Scorecard (§11.4)

The post-build scorecard and `agentic-eval` are complementary, not redundant:

| Scope | Post-Build Scorecard (§11.4) | `agentic-eval` |
|---|---|---|
| Spec integrity | C — Joule config validation (schema, naming) | C1/C2 — Spec Completeness + Clarity |
| Spec-to-code alignment | B — generation drift (spec_sha256 binary match) | C6 — broader accumulated drift across the sprint |
| SAP standards | Indirect (routing quality) | C9 — explicit CAP + Fiori + Hyperspace |
| Code quality | Not covered | C10 — AI anti-patterns |
| Design rationale | Not covered | C5 — design thinking, alternatives evaluated |

Category B (generation drift) is a precise, fast, automatable binary check on whether agent
code was re-generated after a spec edit. C6 catches the broader structural drift that accumulates
across an entire sprint of iterative development. Both are needed.

### Installation

The skill is part of the CIS developer efficiency tools repo (also listed as a Starter Kit
component in [Chapter 9, section 9.4](reusable-components-560fb10.md)).
**Contacts / owners: Apurba Ghosh, Saurabh Kejriwal.**

```bash
# Project-level (one engagement)
npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval

# Global (all projects)
npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval -g
```

Invoke as `/agentic-eval [project-path] [--html] [--word]`. The `--html` flag generates
a self-contained report with SAP Horizon styling and collapsible per-criterion cards. The
`--word` flag generates a Word document in the Porsche sample report style. Both formats
are suitable as FDE Gate or EAC acceptance evidence artifacts.

---

## 11.14 Connecting Evaluation to the Broader Concept

Evaluation is not an isolated activity — it is the feedback mechanism that makes the
rest of the concept operational:

| Concept chapter | How evaluation connects |
|---|---|
| [Ch 01 — Agent Quality Framework](agent-quality-framework-86a2b10.md) | Success rate and diagnostic metrics from aeval are the concrete implementation of the three evaluation dimensions |
| [Ch 02 — Supervisor Agent Pattern](supervisor-agent-pattern-b40ee37.md) | Online eval complements supervisor real-time monitoring; they cover different failure detection windows. Feedback loop Case 3 prevents incorrect HITL overrides from corrupting the agent spec. |
| [Ch 04 — Precision Guardrails](precision-guardrails-3841e39.md) | Boundary condition tests (at threshold, above threshold, unit-confused inputs) must be in the eval suite. Guardrail tests verify Layer 3 escalates correctly, not just that the agent produces a number. Post-build scorecard Category E validates MUST-rule compliance before every deployment. |
| [Ch 08 — Production Operations](production-operations-324bb02.md) | Rollback decisions (§8.3) are triggered by metric regressions; online eval generates the trend data that confirms a regression is real and correlated with a deployment event |
| [Ch 08 §8.10 — TR1–TR14](production-operations-324bb02.md) | TR9 mandates both offline and online eval. The sprint integration points in §8.10 align with the Quality Flywheel in §11.9 of this chapter |
| [Ch 09 — Reusable Components](reusable-components-560fb10.md) | `agentic-eval` is a Starter Kit component (§9.4); the C6 Spec-to-Code Alignment criterion reinforces the spec-first delivery discipline the Starter Kit is designed to support |
| [Ch 10 — Token Efficiency](token-efficiency-f8f90a7.md) | `total_tokens` and `total_cost_usd` from eval runs feed the build-time cost pool (§10.3); cost per correct outcome (§10.5) requires both eval results and token data |
| AI Quality Drift Monitor | Dedicated discovery concept operationalising post-deploy semantic drift detection for FDE customer deployments. Builds directly on the online eval capability (§11.11) and the Quality Flywheel (§11.9): the go-live baseline captured during EAC is the reference point; the drift type taxonomy (§11.11) determines the corrective action routed from the drift monitor's alert pipeline |

---

## 11.15 FDE Delivery Implications

### During Discovery

- For each numerical output or high-risk agent action in the TO-BE process flow, define
  at least one boundary condition test case: what input should trigger the guardrail?
  What input should pass? Document these in the Draft Product Design Specs.
- Confirm which SAP calculation services exist in the customer's landscape — they
  determine whether `tool_call` validations against authoritative references are feasible.
- Agree with the process owner what "success" means for each user journey. This
  agreement becomes the `agent_response_validations` in the test case.
- Run the G1–G4 readiness gates (§11.3) against the compiled discovery outputs before
  any agent spec is generated. Gate scores below 0.8 should be resolved — or their
  coaching questions answered — before the team enters the Development phase.
- Run `agentic-eval` (§11.13) against the Discovery output documents (spec, design docs,
  CLAUDE.md) at the end of Discovery. A C1–C5 average below 6/10 or a C7 score below
  "Agent-Ready" should be resolved before Develop kickoff — spec gaps are cheaper to fix
  before code generation begins than after.

### During Develop

- Week 1: create the first test cases (traceless). Even three `agent_response` checks
  on the happy-path user journey are enough to establish the evaluation baseline and
  validate the eval framework setup. If production data is not yet available for agent
  grounding, generate synthetic data using the `/fde-ontology` skill (Project LightSpeed
  Stage 5): it reads the logical schema and uses Claude to produce domain-contextual rows —
  not random placeholders — that can be pushed to HANA Cloud KG. Evaluation runs grounded
  on synthetic data are meaningful quality signals; the same data is replaced with live
  system data in later phases without changing the eval suite structure.
- Keep test cases in version control alongside the agent workflow. Treat a failing test
  case the same way you would treat a failing unit test: do not proceed to the next sprint
  phase with known failing checks.
- The eval report (HTML format from `aeval`) is the evidence package for the FDE Gate.
  Produce it before the Week 6 review, not during it.
- Run the post-build scorecard (§11.4) before the Week 9 deployment gate. A generation
  drift failure (Category B) at Week 9 is recoverable but disruptive; catching it earlier
  during iterative development avoids last-minute blockers.
- Run `agentic-eval` (§11.13) against the full project repo in the Weeks 7–9 window —
  before EAC, not during it. C6 (Spec-to-Code Alignment) is the metric to watch:
  gaps must be resolved or explicitly accepted and documented in the handover package.
  The HTML or Word export is suitable as EAC acceptance evidence.
- A2A endpoint exposure (TR1) should be implemented early — the eval harness invokes
  agents via A2A. An agent without an A2A endpoint cannot be evaluated by `aeval` and
  therefore cannot satisfy TR9.

### During Productization

- The offline eval suite must be running in CI (TR9) before Sirius productization tasks
  begin. The QPA test strategy references eval coverage.
- The online eval configuration (requirements JSON, lookback window, sampling rate)
  must be part of the production handover package — the customer's IT admin and CIS T&O
  need to know that continuous monitoring is active and what it is monitoring.
- Configure the post-deployment feedback loop (§11.5) outcome horizons to match the
  business process: financial postings may require a 24-hour confirmation window;
  supply chain decisions may require longer. Outcomes that return `outcome_confirmed_bad`
  escalate to CRITICAL and require human review before any spec or memory update is applied.
- Re-run the full eval suite against any model version change (TR9 implication). A model
  update that degrades success rate by more than 5 percentage points from baseline is a
  rollback trigger (Chapter 8 §8.3).

---

## 11.16 Summary

| Topic | Key principle |
|---|---|
| Three-phase lifecycle | Pre-build (G1–G4 gates) → post-build (6-category scorecard) → post-deployment (feedback loop + online eval). Evaluation is not a final-sprint activity |
| G1–G4 readiness gates | Four scored gates verify discovery completeness before spec generation. ≥ 0.8 = proceed; 0.5–0.8 = proceed with coaching questions; < 0.5 = return to discovery |
| Post-build scorecard | 6-category hybrid evaluation (deterministic + LLM-graded). Category B generation drift is a blocking failure: spec_sha256 mismatch means the deployed agent does not implement its spec |
| Spec quality evaluation (agentic-eval) | 11+1 criteria framework measuring spec completeness, AI-readiness, code-spec alignment, MCP utilization, and SAP standards. Two run points: end of Discovery (C1–C7 spec gate) and pre-EAC (full 11+1-criteria sweep). C6 Spec-to-Code Alignment (15% weight) is the productization risk indicator |
| Post-deployment feedback loop | Two parallel paths: spec path (human-gated, HITL overrides → spec edits) + memory path (autonomous LLM analyst + auditor). Four learning cases govern all outcomes |
| Operator training issue (Case 3) | When agent was correct and human overrode incorrectly: do NOT update the spec. Flag for operator training and track separately. Failure to handle Case 3 corrupts agent behavior over time |
| Testing vs. evaluation | Testing = ongoing defect finding; evaluation = milestone-based quality comparison. Both required, different tools |
| TR9 compliance | Offline eval in CI (before each release) + online eval in production. Not optional. Start offline eval at sprint week 1 |
| Success rate | Primary release gate: binary (all validations pass = 1). All other metrics are diagnostic |
| Test case as deliverable | YAML test suite is a sprint artifact; version-controlled, reviewed at milestones, in CI before productization |
| Traceless mode | Start evaluating at sprint week 1 without OTel setup. Add trace-enabled eval when instrumentation is complete (weeks 4–6) |
| Online eval | LLM-as-judge scoring of real production sessions. Complements supervisor (real-time) with statistical trend detection (across sessions). Complements feedback loop (aggregate trends vs. event-driven learning) |
| Quality Flywheel | Weeks 1–3: traceless + happy-path → Weeks 4–6: trace-enabled + tools → Weeks 7–9: full suite + CI → Weeks 10–12: online eval active |
| Synthetic test data | When production data is unavailable (typical in weeks 1–6), `/fde-ontology` (LightSpeed Stage 5) generates domain-contextual synthetic data from the logical schema using Claude. Enables meaningful evaluation from week 1 without live system access; replaced with live data in later phases without changing the eval suite |
| A2A first | TR1 (A2A endpoint) is a prerequisite for eval harness integration. Build A2A exposure early to unblock evaluation |
