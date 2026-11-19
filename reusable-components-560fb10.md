[//]: # (loio: 560fb10)

# Chapter 9: CIS Delivery Efficiency — Reusable Components and Reference Implementations

## 9.1 Why Reusability Is a First-Class Design Concern

CIS operates across multiple Industry AI FDE engagements simultaneously, and the number
grows as the Autonomous Enterprise vision scales. Without intentional reuse, each
engagement rebuilds the same infrastructure from scratch: three supervisor workflows,
an audit trail schema, environment migration checklists, a rollback procedure. The
build effort is real; the testing effort is real; and a freshly assembled Workflow B
has less reliability evidence behind it than one that has run in production across
four previous engagements.

Reuse is not just about efficiency. It is also about quality accumulation: production
incidents discovered in one engagement feed back into the shared reference implementation
so that every subsequent engagement benefits. This is the operational expression of the
continuous improvement loop described in Chapter 6 section 6.6, applied not just to
individual deployed agents but to the CIS delivery patterns themselves.

This chapter maps every design component from Chapters 1–8 to its reusability profile:
what stays the same across engagements, what requires per-engagement configuration,
and what is genuinely engagement-specific. It then defines the **CIS Starter Kit** —
the set of reference implementations that CIS T&O should maintain, version, and provide
to every FDE team as a starting point.

---

## 9.2 The Reusability Stack

The concept components organize into three layers. The distinction matters because each
layer has a different delivery model: the first is inherited, the second is configured,
the third is built.

### Layer 1 — Deploy As-Is

Zero per-engagement content. These components are structurally and logically
domain-agnostic: they operate on telemetry signals, schema definitions, process
checklists, and design rules that do not change regardless of which Industry AI domain
is being served or which worker agent is being supervised.

CIS T&O builds these once, maintains them as versioned reference artifacts, and provides
them to FDE teams to inherit unchanged.

### Layer 2 — Parameterize and Deploy

Fixed skeleton; per-engagement content in defined slots. The workflow structure, routing
logic, and data schemas are the same in every engagement. What varies are values
(threshold calibrations, recipient IDs, connection credentials), prompts (LLM critic
system prompts), and rules (the specific verification logic for this agent's output type).

CIS T&O maintains the skeleton. The FDE team fills the per-engagement slots during
Discovery and Execute, and documents the resulting configuration in the production
handover package.

### Layer 3 — Engagement-Specific

Cannot be lifted across engagements. This is the worker agent's business logic,
the process KPIs that define what the agent is optimizing for, the verification rules
specific to this output type, and the threshold values agreed with the process owner.
These require genuine per-engagement design work; attempting to reuse them from a prior
engagement introduces the wrong assumptions.

The total build scope of a new FDE engagement is the Layer 3 content plus the
parameterization work for Layer 2. Layer 1 is not in scope for the FDE sprint.

---

## 9.3 Component-by-Component Analysis

The following table classifies every significant design component from Chapters 1–8.

> **Layer denotes reusability profile, not current availability.** Throughout this
> chapter (including the §9.10 summary), a component's Layer is its *reusability profile*
> — deploy-as-is vs. parameterize vs. engagement-specific — which is independent of
> whether a forkable reference artifact exists today. Several Layer 1/2 reference
> artifacts — Supervisor Workflows B and C, the shared-state-store DDL, and the APM
> archetype config — are **design targets** not yet published as standalone, forkable
> artifacts. The authoritative statement of what is available now is the **§9.4 status
> table**; read every "Deploy As-Is" / "Fork and configure" entry below against it.

### Layer 1 — Deploy As-Is

| Component | Chapter ref | What is fixed | Parameterization surface | Maintenance owner |
|---|---|---|---|---|
| Supervisor Workflow C (Support Relay) | Ch2 §2.3.3, Ch5 §5.5 | Full workflow: sanitization rules, SAP Cloud ALM ITSM Adapter API call, IT admin notification | Customer tenant ID (from environment config, not hardcoded) | CIS T&O |
| Audit trail schema — base fields | Ch4 §4.7 | All 16 fields: timestamp, agent_id, agent_version, correlation_id, input_state, agent_output, stage verdicts, human_decision, human_modified_value, committed_value, delta | Table name prefix per engagement | CIS T&O |
| Audit trail schema — cost extension | Ch8 §8.5 | Four additional fields: input_tokens, output_tokens, total_tokens_invocation, model_used | None | CIS T&O |
| Telemetry anomaly rule set (Workflow B rule logic) | Ch2 §2.3.2 | All six rule types: error rate, tool call count per invocation, response latency, human override rate, verification rejection rate, identical input pattern | None — rule types are domain-agnostic; threshold values are Layer 2 | CIS T&O |
| Sanitization rules table (what to strip vs. retain in trace exports) | Ch2 §2.5.3 | Field-by-field retain/strip classification | None | CIS T&O |
| Quality signal escalation structure (Level 0–3) | Ch1 §1.6 | Four-level structure: automated → monitored → supervised → blocked; and the logic distinguishing each level | Specific threshold values and recipient IDs are Layer 2 | CIS T&O (structure) |
| Version stack discipline (5 components) | Ch8 §8.2 | Definition of the five-component version stack; change classification table (Low / Medium / High / Very High risk) | Applied per engagement to that engagement's specific component versions | CIS T&O |
| Supervisor fail-safe rule | Ch2 §2.7 | If supervisor unavailable, worker routes all outputs to sapTaskCenter — never to autonomous write-back | None | CIS T&O |
| Three things never to copy between environments | Ch8 §8.6 | Credentials, dev/test thresholds, agent identities — all must be re-provisioned per environment | None | CIS T&O |
| When NOT to use multi-agent (decision table) | Ch7 §7.1 | Full decision table with six rows; criteria for single agent vs. multi-agent | None | CIS T&O |
| Multi-agent topology selection table | Ch7 §7.3 | Full comparison matrix: Topology 1 / 2 / 3 across five dimensions | Topology choice is per engagement | FDE team (guided by table) |
| Three pre-migration checklists (dev→test, test→EAC, EAC→prod) | Ch8 §8.8 | All checklist items across three transitions | None — items are the same for every engagement | CIS T&O |
| Partial failure handling rules in multi-agent (atomic commit principle) | Ch7 §7.5 | Rule: partial approval is not approval; all workers or none | None | CIS T&O |
| Prompt injection defense system prompt template | Ch5 §5.7 | Boilerplate instruction text | Role description at the opening line | FDE team |
| Pro-code decision table | Ch5 §5.9 | Full table: which requirements are low-code-sufficient vs. require pro-code | None | CIS T&O |

### Layer 2 — Parameterize and Deploy

| Component | Chapter ref | Fixed skeleton | Parameterization surface | Maintenance owner |
|---|---|---|---|---|
| Supervisor Workflow B (Telemetry Monitor) — full workflow | Ch2 §2.3.2, Ch5 §5.5, Ch8 §8.5 | Schedule trigger, HTTP Request to SAP Cloud ALM + AI Core usage API, Code node rule structure, routing logic, IT admin alert path, cost anomaly checks, Workflow C trigger | Anomaly threshold values (calibrated from EAC baseline); schedule interval; Task Center recipient IDs | CIS T&O (template); FDE team (calibration) |
| Supervisor Workflow A — routing skeleton | Ch2 §2.3.1, Ch5 §5.5 | Webhook trigger, If/Switch routing (BOTH CONFIRM / BOTH REJECT / DISAGREE / low confidence), sapAgent approval signal back to worker, Task Center escalation path, audit trail write | Code node verification rules (engagement-specific); LLM critic systemPrompt; approval threshold value | CIS T&O (skeleton); FDE team (Code node + critic prompt) |
| Supervisor shared state store schema | Ch2 §2.4.3, Ch7 §7.7 | Baseline table structure (rolling metrics per agent); Pending Set table structure for multi-agent Topology 3 | Table name prefix per engagement; initial state is always empty per environment | CIS T&O (DDL template) |
| Four-layer guardrail stack structure | Ch4 §4.3 | Layer sequence (Pre-Check → Output Validation → Human Approval → Audit Trail); which layers are mandatory vs. optional per output risk level | Layer 2 reference calculation service (which SAP service backs it); Layer 3 threshold values; Layer 1 constraint source | FDE team (configured with process owner for thresholds) |
| sapTaskCenter customAttributes pattern | Ch4 §4.6, Ch5 §5.6 | Six-attribute set: proposed value, reference value, deviation %, agent confidence, stage verdicts, correlation ID | Attribute display labels (e.g., "Batch quantity" vs. "Pricing recommendation"); recipient IDs; priority mapping | FDE team |
| Worker agent base pattern (n8n workflow template) | Ch5 §5.4 | sapOrchestration + sapMcpClient + verification node structure; routing to supervised (sapAgent callback) or unsupervised (inline) | Agent task definition in systemPrompt; SAP system connections; specific MCP tools wired | FDE team |
| Three-dimension quality framework (evaluation design) | Ch1 §1.2 | Framework structure (Technical / Process KPI / Business Outcome); Dimension 1 metric types and their Goodhart's Law guard | Dimension 2 process KPIs (engagement-specific, defined in TO-BE process flow); Dimension 3 business outcome metrics | FDE team (with process owner for Dimensions 2–3) |
| Eval test suite structure | Ch1 §1.3 | Scenario type categories (high-freq/low-risk, low-freq/high-risk, exception, novel); acceptance threshold approach per category | Specific test cases; acceptance threshold values; test data strategy | FDE team |
| Confidence-based routing pattern | Ch4 §4.8 | Pattern: confidence field in sapOrchestration outputSchema; If node routing on confidence threshold | Confidence threshold value | FDE team (with process owner) |
| Cost monitoring extension (Workflow B) | Ch8 §8.5 | Token anomaly check logic; runaway detection (5× p99 threshold); daily budget alert; cost-per-committed-output trend logic | Daily budget value; runaway spike threshold; alert routing | CIS T&O (template); customer IT admin (budget values) |
| Environment-agnostic configuration pattern | Ch8 §8.7 | Naming convention: `{{agent-name.parameter-category.parameter-name}}`; separation of BAIP credential objects vs. configuration parameters | Actual parameter names and values per engagement | CIS T&O (convention); FDE team (values) |
| Rollback procedure template | Ch8 §8.3 | Trigger criteria (human override rate, rejection rate, KPI degradation, format failures); rollback order (threshold → model → prompt → workflow); state continuity rules (audit trail immutable, baseline reset) | Authorization path per topology; specific component versions being rolled back | CIS T&O |
| Cross-worker consistency check pattern types | Ch7 §7.5 | Check categories: quantity cross-consistency, unit consistency, timeline consistency, arithmetic cross-check | Specific field names, quantities, and tolerance values for this multi-agent set | FDE team |
| Severity tier table and operating rhythm | Ch6 §6.5 | Four severity tiers (Critical / High / Medium / Low) with response target structure | Actual SLA commitments (agreed per contract); T&O team contact details | CIS T&O (with contract alignment) |
| Continuous improvement loop | Ch6 §6.6 | Loop structure: production findings → categorize → in-place fix vs. build improvement vs. new sprint; iteration pattern analysis signal types | Finding categories and escalation paths for this engagement | CIS T&O |

### Layer 3 — Engagement-Specific

| Component | Chapter ref | Why it cannot be reused | Who authors it |
|---|---|---|---|
| Worker agent business logic | Ch5 §5.4 | The agent's task, SAP system connections, tools, and goal state are unique to this use case and customer | FDE Application Engineer + Domain PM |
| Process KPIs (Dimension 2 of quality framework) | Ch1 §1.2 | Defined in the TO-BE process flow during Discovery; entirely domain- and customer-specific | FDE team + process owner |
| Workflow A Code node verification rules | Ch2 §2.3.1, Ch4 §4.5 | Specific range checks, unit checks, tolerance values apply only to this agent's output type | FDE Application Engineer |
| LLM critic systemPrompt | Ch2 §2.3.1, Ch4 §4.5 | What a "correct" output means is different for a batch release quantity vs. a pricing recommendation | FDE Application Engineer |
| Human approval threshold values | Ch4 §4.6 | Business decision agreed with process owner in Discovery; not derivable from a template | FDE team + process owner |
| Authorization matrices (all agents) | Ch3, Ch7 §7.6 | Scoped to specific SAP systems and entities this engagement accesses; differs per customer and per worker | FDE team + customer IT admin |
| Anomaly detection threshold values | Ch2 §2.6 | Calibrated from EAC baseline observation specific to this agent's behavior; cannot be borrowed from another agent's deployment | CIS T&O (post-EAC calibration) |
| Regulatory audit trail retention period | Ch4 §4.7 | Confirmed with customer's compliance team; jurisdiction- and industry-specific | Customer compliance team |
| Cross-worker consistency check logic | Ch7 §7.5 | Specific field names, quantity relationships, and timeline logic for this multi-agent set | FDE team |
| Business outcome metrics (Dimension 3) | Ch1 §1.2 | Financial and operational results tied to this customer's specific value case | FDE team + Value Advisor |
| Eval test cases and test data | Ch1 §1.3 | Test scenarios must cover this agent's boundary conditions and edge cases | FDE team |
| LLM critic test suite | Ch2 §2.8 | The critic prompt's quality is engagement-specific; its test suite must match its content | FDE team |

---

## 9.4 The CIS Starter Kit

The CIS Starter Kit is the versioned set of Layer 1 and Layer 2 artifacts that CIS T&O
maintains and provides to every FDE engagement as a starting point. It is not a
monolithic package — FDE teams take what applies to their engagement's scope and
leave what does not. A lean engagement that does not require the full supervisor stack
does not need to inherit Workflow B or C; a single-agent deployment without numerical
write-back does not need the audit trail extension.

> **Status — component-level**:
>
> | Component | Status | Available where |
> |---|---|---|
> | **Workflow A skeleton** (`cis-supervisor-workflow-a-skeleton`) | PoC in progress | `concept/proof-of-concept-plan.md` — Workflow A PoC is the first build step |
> | **Workflow B reference** (`cis-supervisor-workflow-b-ref`) | Design target | Not yet a standalone artifact; no supervisor workflow exists in the §9.5 LightSpeed catalog either — build per Ch2 §2.3.2 / Ch5 §5.5 patterns |
> | **Workflow C reference** (`cis-supervisor-workflow-c-ref`) | Design target | Not yet a standalone artifact; patterns from §2.5.3 and Ch5 §5.10 |
> | **Worker agent base** (`cis-worker-agent-base`) | Available now | LightSpeed `references/supply-chain-risk-agent/` and `references/service-technician-agent/`; cookbook `templates/supply-chain-risk-agent/` |
> | **Audit trail DDL** (`cis-audit-trail-ddl.sql`) | Available now | Schema per Ch4 §4.7 / Ch8 §8.5; the audit-trail table is provisioned by the LightSpeed BTP Terraform modules (`infra/btp/`, see §9.5) — parameterize per engagement |
> | **Shared state store DDL** (`cis-shared-state-store-ddl.sql`) | Design target | DDL schema specified in Ch2 §2.4.3 / Ch7 §7.7; standalone file not yet published |
> | **Pre-migration checklists** (`cis-pre-migration-checklists.md`) | Available now | Ch8 §8.8 templates; copy and fill per engagement |
> | **Config naming convention** (`cis-config-naming-convention.md`) | Available now | Ch8 §8.7 — derive from documented pattern |
> | **Rollback runbook template** (`cis-rollback-runbook-template.md`) | Available now | Ch8 §8.3 — template with placeholders |
> | **Sanitization rules** (`cis-sanitization-rules.md`) | Available now | Ch2 §2.5.3 table |
> | **Security checklist** (`cis-security-checklist.md`) | Available now | Ch5 §5.7 |
> | **APM archetype config** (`apm.yml` per domain) | Design target | Structure defined in §9.4 "Starter Kit Distribution via APM" (an `apm.yml` declaring the MCP servers, skills, and `CLAUDE.md` context per engagement archetype); exemplar: `vs-template-apm` (Ch10 §10.3). Not yet a distributed artifact — see APM advisory below |
>
> **Summary**: worker agent and schema artifacts are usable today via LightSpeed; the supervisor reference workflows (A, B, C) are the build priority. Teams forking LightSpeed components as an interim measure should document the divergence from the eventual Starter Kit version in their productization package.

### Starter Kit Contents

**Joule Studio Reference Workflows** (maintained in a CIS-internal Joule Studio project,
version-controlled, tested against the current BAIP platform version):

| Artifact | Description | Fork and configure |
|---|---|---|
| `cis-supervisor-workflow-b-ref` | Workflow B (Telemetry Monitor) with full rule set and cost extension; no engagement-specific content — **design target, not yet a standalone forkable artifact (see §9.4)** | When published: fork; set anomaly thresholds after EAC baseline; configure recipient IDs |
| `cis-supervisor-workflow-c-ref` | Workflow C (Support Relay) — **design target, not yet a standalone forkable artifact (see §9.4)** | When published: fork; verify tenant ID injection from environment config |
| `cis-supervisor-workflow-a-skeleton` | Workflow A with routing structure, webhook trigger, sapAgent callback, Task Center escalation, audit trail write; Code node placeholder for verification rules | Fork; author the Code node verification rules and LLM critic systemPrompt for this engagement |
| `cis-worker-agent-base` | Worker agent base pattern with sapOrchestration, sapMcpClient, verification routing, and supervised/unsupervised path | Fork; define task systemPrompt, wire SAP system connections, add MCP tools |

**Workflow A scoping: one skeleton per output type, not per agent**

The Workflow A skeleton is forked per output type — not per worker agent and not per
engagement. Two workers that produce structurally similar outputs (e.g., two pricing
agents serving different markets, or two batch release agents for different plants) can
share one Workflow A instance: the Code node verification rules and the LLM critic
system prompt are the same because the output type is the same; threshold values that
differ between workers are looked up by `agent_identity` in the Code node rather than
duplicated into separate workflows.

Sharing breaks down when workers produce fundamentally different output types —
different units, different validation logic, different reference calculation services.
At that point a branching Code node becomes harder to test and maintain than two
separate, simpler Workflow A instances. The rule of thumb: if the Code node rules
would be identical for two workers, share the workflow; if they diverge significantly,
create a separate instance.

In multi-agent deployments (Chapter 7, Topology 3), the shared supervisor naturally
handles multiple workers of different output types by differentiating verification by
`agent_identity`. The supervisor sits between the workers and the orchestrator: it
verifies each worker's output independently, tracks which callbacks have arrived via
the Pending Set table in the shared state store, and releases the combined approved
set to the orchestrator only when all workers have been verified. The orchestrator
never receives an unverified worker output.

**Schema Artifacts** (DDL scripts; the audit-trail DDL is available now, the shared-state-store DDL is a design target per §9.4):

| Artifact | Description |
|---|---|
| `cis-audit-trail-ddl.sql` | HANA Cloud DDL for the append-only audit trail table; includes all base fields and cost extension fields from Chapter 4 section 4.7 and Chapter 8 section 8.5 |
| `cis-shared-state-store-ddl.sql` | **Design target (see §9.4):** HANA Cloud DDL for the supervisor shared state store (baseline table + Pending Set table for multi-agent Topology 3). The schema is specified in Ch2 §2.4.3 / Ch7 §7.7; a standalone DDL file is not yet published |

**HANA Cloud provisioning**: both DDL artifacts target HANA Cloud (not HANA on-premise). HANA Cloud is provisioned as a BTP service in the engagement's BTP sub-account. In Project LightSpeed, `pilot.yaml` handles HANA Cloud provisioning as part of the BTP desired-state model — `cookbookctl apply` provisions the HANA Cloud instance, schema, and all required tables in a single operation. For engagements not using LightSpeed, provision HANA Cloud manually via the BTP cockpit: allocate the HANA Cloud free tier or production plan in the appropriate sub-account, create the schema, and run the DDL scripts. Estimated storage for audit trail and shared state store: plan for 10–50 MB per month of production operation per agent; the append-only audit trail grows continuously — confirm a retention policy with the customer compliance team (Chapter 4 §4.7) and implement scheduled archiving after 90 days.

**Process and Configuration Artifacts**:

| Artifact | Description |
|---|---|
| `cis-pre-migration-checklists.md` | Three environment transition checklists (Chapter 8 section 8.8); used unchanged for every engagement |
| `cis-config-naming-convention.md` | Configuration parameter naming convention and environment-agnostic wiring pattern (Chapter 8 section 8.7) |
| `cis-rollback-runbook-template.md` | Rollback procedure template with placeholders for engagement-specific component versions and authorization contacts |
| `cis-sanitization-rules.md` | Sanitization rules table (Chapter 2 section 2.5.3); included by reference in the Sirius security concept documentation |
| `cis-security-checklist.md` | Per-workflow security checklist (Chapter 5 section 5.7); used as a review gate before every environment transition |

**Evaluation Skills** (Claude Code skills, installable via `npx skills add` or `agpm`):

| Skill | Description | Install command |
|---|---|---|
| `agentic-eval` | 11+1 Agentic Engineering Spec Quality Assessment; evaluates spec completeness, AI-readiness, code-spec alignment (C6), MCP utilization, and SAP standards compliance (C9). Run at end of Discovery and pre-EAC. HTML and Word report export. | `npx skills add https://github.tools.sap/CISDeveloperEfficiencyTools/developer-efficiency-tools.git --skill agentic-eval` |

See [Chapter 11, section 11.13](agent-evaluation-8cabb2d.md) for the full criteria framework, score thresholds, and sprint timing guidance. Skill owners: Apurba Ghosh, Saurabh Kejriwal.

### Starter Kit Governance

The Starter Kit is owned by CIS T&O and governed as a product, not as documentation:

- **Version**: each release of the Starter Kit is tagged. FDE teams record which Starter
  Kit version their engagement forked from.
- **Update cadence**: the Starter Kit is updated when BAIP platform changes affect the
  reference workflows, when a production incident reveals a gap in the rule set or
  schema, or when a completed engagement produces a generalizable improvement.
- **Improvement pipeline**: production findings from the continuous improvement loop
  (Chapter 6 section 6.6) that are generalizable across engagements are proposed as
  Starter Kit updates. CIS T&O reviews and merges them.
- **Deprecation**: when BAIP node APIs or Joule Studio behavior changes, the Starter
  Kit is updated before the next engagement forks from it. Running a stale fork in
  production without reconciling against the current version is a governance gap.

### Starter Kit Distribution via APM

> **Advisory — APM provenance and use scope**: APM (Agent Package Manager) is a
> **Microsoft-originated, MIT-licensed open-source tool** (github.com/microsoft/apm),
> *adopted* by SAP for FDE build-time use — it is not an SAP-built tool. Governing rule
> (SDOL-005 / PPMS): OSS compliance obligations attach to components **shipped to
> customers**, not to CLI tools that run only on SAP dev workstations or CI. `apm install`
> is a build-time operation that writes config into harness directories and does not
> appear in the deployed customer artifact, so **build-time use on SAP-internal CI is
> fine**. Customer-tenant deployment (Topology A — customer-managed BTP sub-account) is
> **uncleared until a PPMS entry is confirmed**; the trigger is whether the `apm` binary
> or its generated output is bundled into a customer deliverable (Docker image,
> customer-operated CI, MCP server package). No confirmed SAP Software Hub / PPMS
> registration was found — treat any "Software Hub Adequate / Permitted" claim as
> unverified until the record is cited. For a binding use-scope ruling, contact
> **SAP_CIS_ATQ@sap.com**. Track PPMS confirmation as a Starter Kit readiness item before
> enabling any customer-facing distribution path.

The CIS Starter Kit configuration — which MCP servers, skills, and `CLAUDE.md` context
apply to a given engagement type — should be distributed using APM (Agent Package
Manager), a per-project agentic setup tool. An `apm.yml` file per engagement archetype
(e.g., asset management, regulated manufacturing) declares the correct tool set and
skills; `apm install` deploys them to each engineer's environment; `apm compile`
generates the project `CLAUDE.md`. This gives every FDE engineer on the squad an
identical, version-locked starting point and avoids the global-install anti-pattern where
each engineer's session starts with a different tool set.

The `apm.yml` template for each engagement archetype is a Starter Kit artifact, owned
and versioned by CIS T&O alongside the reference workflows. See
[Chapter 10, section 10.3](token-efficiency-f8f90a7.md) for the per-project agentic
setup pattern and the efficiency rationale behind it.

### Customer-Managed Skill Registry

Beyond the Starter Kit's cross-engagement reference artifacts, high-maturity CIS
deployments benefit from a **customer-managed Skill Registry** — a governed store of
reusable agent behaviors specific to that customer's SAP landscape:

| Governance feature | What it provides |
|---|---|
| **Version pinning** | Skills are versioned; agents pin to a specific skill version rather than pulling the latest |
| **Approval workflows** | New or updated skills require review and sign-off before they can be called in production |
| **Cross-team distribution** | Skills developed for one agent are discoverable and reusable by other agents in the same tenant — turning project-local patterns into tenant-wide assets |
| **Compounding knowledge** | Every production fix, edge case, and workaround feeds back into the registry as an updated skill; operational learning accumulates rather than being lost |

The Skill Registry sits at the boundary between the Starter Kit (cross-engagement
artifacts maintained by CIS T&O) and engagement-specific Layer 3 content. Skills
general enough to apply to multiple agents within a single customer deployment belong
in the registry; skills specific to one workflow's logic do not.

**FDE delivery implication**: the Skill Registry is a productization and handover
consideration. When the FDE team hands over to the customer's IT admin and CIS T&O,
the registry is one of the governance artifacts the customer IT team should own and
operate. Include the Skill Registry governance model — who approves new skills, who
manages versions, how the registry is backed up — in the productization package
alongside the agent authorization matrix and the T&O operational runbook.

---

## 9.5 Project LightSpeed: CIS Reference Implementation

Project LightSpeed (`github.tools.sap/CIS-Solution-Acceleration/Project-Lightspeed`)
is the current, built CIS component library. Where §9.4 describes a specification of
what the Starter Kit should contain, LightSpeed is what exists today. FDE teams should
treat the LightSpeed component catalog as the authoritative starting point; Starter Kit
artifacts that do not yet exist in a standalone CIS project can be bootstrapped from
the corresponding LightSpeed component.

### Component Catalog

| Component | Location in repo | What it provides |
|---|---|---|
| OTel/logging middleware | `references/middleware/` | Drop-in Python package; 2-line setup; graceful degradation (Cloud Logging, env-OTLP, logging-only fallback); works in local dev, SAP Cloud Logging, and sovereign regions |
| Supply-chain risk reference agent | `references/supply-chain-risk-agent/` | Reference agent implementation with live S/4HANA tooling; starting point for supply-chain domain engagements |
| Service technician reference agent | `references/service-technician-agent/` | Reference agent implementation with field service tooling; starting point for asset management domain engagements |
| Maintenance planner example agent | `agents/maintenance-planner/` | Complete example agent with spec, code, and Joule capability bundle |
| Procurement assistant example agent | `agents/procurement-assistant/` | Complete example agent with spec, code, and Joule capability bundle |
| BTP Terraform modules | `infra/btp/` | Reusable Terraform modules: foundation stack, identity stack, joule stack; parameterized for BTP sub-account provisioning |
| S/4HANA entity catalog | `catalog/s4_catalog.sqlite` | Offline SQLite catalog of S/4HANA canonical objects, attributes, and APIs; enables entity grounding without an EKX connection at build time |
| A2A Agent Toolkit | `toolkits/a2a-agent-toolkit/` | Agent scaffolding toolkit in TypeScript (CAP + Express) and Python (FastAPI + LangGraph) implementing the A2A inter-agent protocol |
| Discovery/build skills | 30+ skill files across pipeline stages | Reusable Claude Code skills covering all 13 pipeline stages; each skill is independently usable on other projects |
| Engagement config template | `pilot.yaml.example` | Template for the BTP desired-state file (park/resume/adopt patterns); copy and parameterize per engagement |

The OTel middleware, Terraform modules, and A2A toolkit are drop-in components that
can be adopted into any CIS engagement regardless of whether the full LightSpeed
pipeline is used. The S/4HANA entity catalog (`s4_catalog.sqlite`) is particularly
useful as a fallback when EKX connectivity is unavailable at build time.

**RAG ingest pipeline** (`fde_rag_ingest.py`): Project LightSpeed now includes a
standard RAG ingest component as a stage 9 output. This component handles document
chunking, embedding generation, and vector store population for BTP-hosted vector
stores. It is parameterized for the engagement's grounding corpus (document types,
chunking strategy, embedding model) and is a reusable Layer 2 component — the
pipeline structure is fixed; the corpus configuration is per-engagement. For any
FDE agent that uses RAG grounding (`sapOrchestration groundingConfig`), the LightSpeed
RAG ingest pipeline should be the starting point rather than a custom implementation.

**Context transfer schema** (`fde_context_transfer.py`): LightSpeed also generates a
standard context transfer artifact that documents the agent's knowledge extraction —
the entity catalog, goal/capability mapping, integration bindings, and process KPIs
— in a structured, machine-readable format. This schema is the bridge between
discovery artifacts and the agent's runtime grounding. It is produced during stages
1–8 of the LightSpeed pipeline and updated throughout the sprint as grounding content
evolves. The context transfer schema is a handover artifact: it documents what the
agent knows and where it learned it, which is required for the Sirius security concept
and for the continuous improvement loop when the grounding corpus is updated.

### Versioned-Skill Distribution Pattern

The book-of-fde and LightSpeed both use the same distribution model for Claude Code
skills: a skill is a Markdown file (`skill-name.md`) stored in a git repository.
Installation is a single `gh api` call against `github.tools.sap`; contribution back
is a PR. A pre-commit version-bump hook enforces a version increment on every skill
change, ensuring the version header in the skill file always reflects the current state.

This distribution model is the standard for CIS CoE skill maintenance:

| Operation | Mechanism |
|---|---|
| Install a skill | `gh api` call against the repository; skill file written to `~/.claude/skills/` |
| Contribute a fix or improvement | PR against the source repository |
| Enforce version consistency | Pre-commit hook increments version on every change to the skill file |
| Pin a specific version | Reference the commit SHA or tag in the install command |
| Audit skill contents | Inspect the Markdown file directly — no binary packaging |

### Security Note: Skills Are Customer-Inspectable

Skill files are plain Markdown and are readable by anyone with access to the Claude
session or the repository. This is a feature (auditability) and a constraint
(confidentiality):

- **Skills are appropriate for**: process automation, phase routing, checklist
  enforcement, tool orchestration, and coordination logic that can be shared with
  customers or disclosed in an audit.
- **Skills are NOT appropriate for**: proprietary algorithms, confidential business
  rules, pricing logic, or any logic that must remain confidential relative to the
  customer.
- **For IP-sensitive logic**: implement in a code-based agent (Python/TypeScript),
  not a skill. The deployed agent binary is not disclosed; the skill file is. These
  serve different confidentiality profiles.

If an FDE team is uncertain whether a piece of logic belongs in a skill or in agent
code, the default is agent code.

### Custom Agentic Solutions CoE Cookbook — Second CIS Reference Library

Alongside Project LightSpeed, the **Custom Agentic Solutions CoE Cookbook**
(`github.tools.sap/<internal-org>/Custom-Agentic-solutions-CoE`) is a second CIS
reference implementation and field delivery resource. Where LightSpeed focuses on the
end-to-end FDE pipeline (13 stages from discovery artifacts to deployed multi-agent
application), the cookbook focuses on the agent delivery primitives: how to scaffold a
BTP-hosted A2A agent, deploy it to CF or Kyma, and wire it into Joule via the
customer-managed (CMJ) orchestrator path (the sovereign/legacy option) — under the SMJ default, a BYOA agent instead registers to the SAP-managed Agent Gateway via A2A.

The two libraries are complementary, not competing. LightSpeed uses and embeds the
cookbook's A2A Agent Toolkit. For most FDE engagements the cookbook provides the
low-level build and deploy patterns; LightSpeed provides the engagement-level
orchestration and pipeline.

**Cookbook structure (key components):**

| Component | Location in repo | What it provides |
|---|---|---|
| A2A Agent Toolkit | `toolkits/a2a-agent-toolkit/` | Claude Code plugin (`sap-a2a-agent-toolkit`, v0.6.0) that scaffolds and deploys A2A agents; drives Checkpoints 1–3 |
| Supply-chain risk agent template | `templates/supply-chain-risk-agent/` | Canonical Python A2A agent starter (four-file shape: `agent.py`, `agent_executor.py`, `app.py`, `__main__.py`) |
| Joule capability skill | `skills/sap-joule-capability/` | 4-file YAML bundle structure, deploy commands, pitfall guidance for CMJ Joule registration |
| BTP deploy recipes | `recipes/02-deploy-btp/` | CF and Kyma deployment patterns with smoke tests |
| Joule wiring recipe | `recipes/03-joule/` | Region availability check, `joule deploy` command, troubleshooting, cleanup |
| Sovereign model gateway | `recipes/optional/sovereign-model-gateway/` | Required for sovereign/regulated regions; OpenAI-compatible gateway as AI Core replacement |
| Observability recipe | `recipes/optional/observe-and-eval/` | Cloud Logging binding, OTel middleware placeholder, region-specific log targets |
| FDE code-first stack reference | `references/fde-code-first-stack.md` | North-star stack (Kyma + A2A v1.0 + SAP Cloud SDK for Python); ownership split; access checklist |
| SwissKnife pointer | `references/btp-joule-a2a-pro-code-agent/` | Pointer to public SAP Architecture Center reference (`SAP-samples/btp-joule-a2a-pro-code-agent`) |
| cookbookctl | Root | CLI coordinator for manifest-driven BTP provisioning; `pilot.yaml` desired-state model |

**Three-checkpoint delivery playbook:**

| Checkpoint | Recipe | Output evidence |
|---|---|---|
| **1 — Local agent** | `recipes/01-scaffold-agent/` | `SendMessage` returns non-error response locally |
| **2 — BTP deploy** | `recipes/02-deploy-btp/` | Agent Card reachable at public URL; smoke test passes |
| **3 — Joule wiring** | `recipes/03-joule/` | `joule list capabilities` shows ACTIVE; Joule routes a real prompt to the agent |

**Field adoption context:** The cookbook is actively used in GDH (Go-to-Market Digital
Hub) deployments, with Kai Mühlbauer's team using it as a customer engagement template.
It was built initially for SovCloud/NS2 deployments where BAIP is not yet available,
and has since been adopted more widely for commercial-region FDE engagements. The
cookbook is a source of truth for **BYOA custom-agent runtime delivery patterns** —
building a custom agent on the customer's BTP subaccount and registering it to the
SAP-managed orchestrator. Note that delivery should be read as **two orthogonal axes,
not a single bundled "CMJ/BYOA" path**: the **orchestrator** is SMJ (SAP-Managed Joule),
the default for new engagements, with CMJ (Customer-Managed Joule) now a deprecated
migration target rather than a delivery choice; the **custom-agent runtime** is either
BYOA (customer subaccount — the standard runtime for productization-intent agents,
registered to SMJ via A2A) or Joule Studio 2.0 (low-code). **BYOA is not a fallback.**
See §5.13 and §6.8 for the delivery-model decision guide.

**Public release track:** The cookbook has an active port to a public SAP OSS repository
(`github.com/SAP/custom-agentic-cookbook`). Keep internal-only URLs, dependencies, and
customer material out of any content intended for the public track.

---

## 9.6 What Parameterization Happens in FDE Discovery

Discovery is where Layer 2 skeletons acquire their per-engagement content. The
following decisions must be made in Discovery — before the Development phase begins — to
ensure the FDE sprint can proceed without scope uncertainty:

| Decision | Where it is documented | Who owns it |
|---|---|---|
| Supervisor scope: which workflows are in scope (A only / A+B / A+B+C)? | Draft Product Design Specs | FDE Domain PM + Discovery Lead |
| Multi-agent or single agent? If multi-agent: which topology (1 / 2 / 3)? | Draft Product Design Specs | FDE Dev Architect |
| Which SAP calculation service backs Layer 2 of the guardrail stack? (Or: does one exist at all?) | Draft Product Design Specs | FDE Application Engineer + P&E Expert |
| Human approval threshold values (absolute and deviation-based) for each numerical output | Draft Product Design Specs | FDE team + process owner |
| Authorization matrix for all agents | Draft Product Design Specs | FDE Dev Architect + customer IT admin |
| Regulatory audit trail retention requirement | Legal agreement plan | Customer compliance team (confirmed during Discovery) |
| Process KPIs (Dimension 2) for the quality framework | TO-BE process flow | FDE Domain PM + process owner |
| Legal agreement path for Workflow C (support relay), if in scope | Legal agreement plan | FDE team + SAP legal |
| Cost monitoring budget values and alert owners post-handover | Production handover plan | FDE Domain PM + customer IT admin |
| Eval test data strategy: synthetic vs. TEA-covered real data | Eval plan | FDE Application Engineer + Data Scientist |

Decisions left unresolved at the end of Discovery become scope risk in the Execute
phase. The threshold values and authorization matrices in particular cannot be
retrofitted — they shape the workflow design and the Sirius security concept
documentation that must be complete before productization.

---

## 9.7 FDE Sprint Scope Implication

The Starter Kit does not eliminate the FDE sprint's build effort — it removes the
infrastructure rebuild that otherwise consumes sprint capacity on work that adds no
differentiation for this customer.

Approximate effort comparison for the supervisor stack (Workflows A, B, C, shared
state store), based on the full-stack path in Chapter 5 section 5.10:

| Activity | Without Starter Kit | With Starter Kit |
|---|---|---|
| Workflow C (Support Relay) | 1–2 days build + test | ~2 hours: fork, configure, verify sanitization in EAC |
| Workflow B (Telemetry Monitor) | 2–3 days build + test | ~1 day: fork, calibrate thresholds post-EAC baseline, test with injected anomalies |
| Workflow A skeleton | 2–3 days build | ~1 day: fork, author Code node verification rules + critic prompt, test routing |
| Shared state store | 0.5 days DDL + deploy | ~1 hour: deploy from DDL template, verify in each environment |
| Audit trail | 0.5 days DDL + deploy | ~1 hour: deploy from DDL template, verify append-only behavior |

These are estimates for planning purposes, not commitments. Actual effort depends on
the complexity of the engagement-specific verification rules in Workflow A and the
availability of a deterministic Layer 2 reference service. The "With Starter Kit"
figures for Workflow B, Workflow C, and the shared state store assume the published
reference artifact; per the §9.4 status table these three are currently **design
targets**, so until they are published the applicable effort is closer to the "Without
Starter Kit" column.

The freed capacity goes into the work that actually differentiates the engagement:
the worker agent's business logic, the evaluation test suite, integration testing, and
the process owner alignment sessions that determine threshold values.

**Important**: the Chapter 5 section 5.10 build sequence still applies when using the
Starter Kit. Fork and configure the supervisor workflows only after the worker agent is
stable — not before. A supervisor configured to govern an unstable worker creates
debugging noise that slows the sprint.

---

## 9.8 Relationship to SAP AI Agent Hub

The Starter Kit is a build-time and operations-time artifact; SAP AI Agent Hub is the
runtime governance layer. The two are complementary:

- The Agent Hub's **Agent Registry** records every deployed agent (workers, supervisors,
  orchestrators) with its identity, version, and authorization scope. The Starter Kit
  provides the artifacts that get registered.
- The Agent Hub's **Agent Observability** surfaces the telemetry that Workflow B
  monitors via the SAP Cloud ALM API. The Starter Kit's Workflow B is the
  consumer of what the Agent Hub's harness produces.
- The Agent Hub's **Secure & Govern** capability manages the threshold configuration
  parameters that the Starter Kit's environment-agnostic configuration pattern exposes
  as named parameters.
- The Agent Hub's **Agent Evals & Assessment** runs the eval test suite against the
  agents the FDE team builds using the Starter Kit worker agent base pattern.

The Starter Kit does not duplicate any of these capabilities. It implements the
CIS-side operational layer — the supervisor agent pattern, the audit trail, the
escalation logic — that operates within and on top of what the Agent Hub provides.

---

## 9.9 Continuous Improvement: From Production Back to the Starter Kit

The continuous improvement loop in Chapter 6 section 6.6 runs within individual
engagements. The Starter Kit adds a second loop that runs across engagements.

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

- A new BAIP telemetry field becomes available that improves anomaly classification
  in Workflow B's Code node → add to the reference implementation
- A production incident reveals that a specific audit trail field was missing and
  required a schema change mid-engagement → add the field to the base DDL
- A pre-migration checklist item is consistently missed across engagements →
  add a more explicit verification step to the checklist
- A new failure mode is observed across two independent engagements in the same
  Industry AI domain → add a new rule to the Workflow B rule set

---

## 9.10 Summary

| Layer | What it contains | Delivery model | Maintained by |
|---|---|---|---|
| Layer 1 — Deploy as-is | Workflow C, audit trail schemas, anomaly rule set, sanitization rules, pre-migration checklists, version stack discipline, decision tables, fail-safe rules | Inherited by FDE team; no sprint effort | CIS T&O |
| Layer 2 — Parameterize and deploy | Workflow B (threshold calibration), Workflow A skeleton (Code node + critic prompt authored per engagement), guardrail stack structure, quality framework, eval structure, rollback template, configuration naming convention | Forked and configured during FDE Execute; parameterization decided in Discovery | CIS T&O (skeleton); FDE team (per-engagement content) |
| Layer 3 — Engagement-specific | Worker agent business logic, process KPIs, verification rules, threshold values, authorization matrices, regulatory requirements | Built from scratch in FDE Execute | FDE team (with process owner and customer IT admin) |

The division of labour is clear: CIS T&O invests in making Layers 1 and 2 excellent
so that FDE teams can invest their sprint capacity in Layer 3. Layer 3 is where the
customer's specific business problem is solved — it is the only layer where CIS cannot
substitute a prior answer for the work of understanding this customer.
