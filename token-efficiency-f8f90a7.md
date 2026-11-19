[//]: # (loio: f8f90a7)

# Chapter 10: Token Efficiency in Industry AI Delivery

## 10.1 Why Token Efficiency Matters

### The Shift from Subsidy to Accountability

The economics of AI development have changed. SAP CTO Philipp Herzig has described the
shift as moving from a "token-subsidy era" — where LLM usage costs were absorbed without
visibility — to a "token-scarcity era" where accuracy, price, and performance must be
optimized together. SAP's Token Consumption Governance Framework, which went live on
17 August 2026 via the Hyperspace LLM Proxy, made this shift concrete for SAP-internal
tooling. Token budgets and consumption plans now apply to developer tools (Claude Code,
Codex, Pi, opencode) at the individual and team level.

This is not a signal to use less AI — it is a signal that token consumption must be
deliberate and measurable. For CIS and FDE teams, deliberate means: knowing where tokens
are spent, having techniques to reduce waste, and framing efficiency not as token count
but as value delivered per token spent.

### Two Cost Pools with Different Owners

The central insight for this chapter is that token cost in Industry AI delivery splits
into two entirely separate cost pools. They have different measurement mechanisms,
different owners, and different optimization levers.

| Cost pool | What it covers | How it is measured | Who owns it |
|---|---|---|---|
| **Build-time** | FDE engineers using Claude Code and AI coding tools during the sprint | Hyperspace LLM Proxy — SAP-internal only | CIS / FDE team |
| **Runtime** | Deployed Industry AI agents processing business transactions for the customer | BAIP AI Units + OpenTelemetry (built into BAIP harness) | Customer (billing) + CIS (operational monitoring) |

> **Note — the runtime pool is itself multi-dimensional.** Shown here as a single line
> for the build-time vs. runtime contrast, the runtime pool decomposes at deployment into
> **three separate, non-consolidatable billing pools — AI Units + BTPEA credits + BDC
> capacity units** — that map to different contract line items. See §10.4 for the
> breakdown and its value-case implications.

**Critical constraint**: the Hyperspace LLM Proxy is an SAP-internal tool. It is not
available for deployed agents running in a customer's BAIP tenant at runtime. Any
efficiency measurement of production agents relies exclusively on the BAIP-native
observability stack: AI Units for commercial billing and OpenTelemetry traces for
operational monitoring (see Chapter 8, sections 8.4 and 8.5 for the operational design).

CIS carries accountability for both pools: build-time efficiency protects the sprint
budget; runtime efficiency protects the customer's value case.

---

## 10.2 SAP's L0–L6 Token Optimization Framework

SAP's internal Token Optimization Cheat Sheet defines seven levels of optimization,
numbered L0–L6. The framework progresses from the cheapest and most impactful
interventions (L0) to governance-layer instrumentation (L6).

| Level | Name | Core action |
|---|---|---|
| L0 | **ELIMINATE** | Use deterministic tools first; ask whether an LLM is necessary at all for this step |
| L1 | **INPUT CONTROL** | Lean prompts; only solution-relevant MCP tools registered; only the context the agent needs for this phase |
| L2 | **OUTPUT CONTROL** | Return diffs not rewrites; set response ceilings; suppress chain-of-thought reasoning where the trace is not needed |
| L3 | **CACHING** | Put stable, rarely-changing content at the beginning of prompts to exploit cache reads; stable `CLAUDE.md` for build-time sessions |
| L4 | **MODEL ROUTING** | Route by complexity: lightweight model for simple tasks, frontier model for hard reasoning; batch and async for non-urgent work |
| L5 | **AGENT CAPS** | Manage session length; use session compaction; write handoff notes at phase boundaries |
| L6 | **GOVERNANCE** | Track cost per correct outcome, not per token; establish measurement baselines; set accountability structures |

The framework applies to both cost pools but with different emphasis:

- For **build-time** (FDE engineers), L0–L5 are practical daily habits and tool
  configuration choices. L6 is the engagement-level measurement question.
- For **runtime** (deployed agents), L0 is the most consequential — agent workflow
  design determines which steps require LLM reasoning and which can use deterministic
  Code nodes. L1–L3 are encoded into the Joule Studio workflow design and the
  `sapOrchestration` configuration.

---

## 10.3 Build-Time Efficiency: FDE Engineer Guidelines

### Current State

No standardized token efficiency guidelines currently exist for FDE teams. The SAP
Token Consumption Governance Framework (L6) is now in place for individual developer
tooling via Hyperspace, but CIS does not yet have:

- Per-engagement or per-project cost tracking (see Q16 in the open questions)
- A prescribed agentic project setup that embeds L0–L5 practices from day one
- A baseline measurement approach that would let engagements compare efficiency over time

This section defines the starting point for building those practices.

### Per-Project Agentic Setup (L1 + L5)

The single highest-impact build-time change is establishing a per-project, reproducible
agentic configuration rather than relying on individual engineers' global tool
installations.

**The anti-pattern**: each engineer installs whatever MCP servers and skills they prefer
globally. Every session starts with a different tool set, different context, and no
shared project conventions. Token waste is structurally guaranteed: the wrong tools are
registered, project context must be re-established from scratch, and there is no common
baseline to measure against.

**The better pattern**: a per-project `apm.yml` (Agent Package Manager, a Microsoft OSS
tool for managing agentic project configurations) that declares:

- Which MCP servers are registered for this project type (not all MCP servers globally)
- Which skills apply to this engagement's workflow
- A project `CLAUDE.md` encoding the project overview, stack constraints, forbidden
  operations, and pointers to ADRs and specs

The `apm install` command deploys the declared tool set; `apm compile` generates the
`CLAUDE.md`. This gives every engineer on the FDE squad an identical, version-locked
starting point. Any improvement one engineer makes to the configuration propagates to
all via a repository update.

The CIS Starter Kit (Chapter 9, section 9.4) is the natural home for an `apm.yml`
template per common engagement type. The `vs-template-apm` pattern — a CIS-maintained
`apm.yml` template for agentic V&S (value & scoping) document creation that bundles the
pre-configured MCP servers and skills for that workflow — is the concrete exemplar of
this mechanism in practice.

**Governance note**: `apm` is a Microsoft-originated, MIT-licensed open-source tool
(github.com/microsoft/apm) adopted by SAP — not an SAP-built tool. No confirmed SAP
Software Hub / PPMS registration could be found, so the previously cited "Adequate /
Permitted (RITM3008136)" status is **pending Software Hub / PPMS confirmation** and must
not be relied on until the record is cited. The governing boundary is build-time-vs-
shipped (SDOL-005 / PPMS): build-time use on SAP-internal CI and developer workstations
is fine because `apm install` writes config into harness directories and does not appear
in the deployed customer artifact. **Customer-environment deployment (e.g., under a CA
where agents run in the customer's tenant) is uncleared until a PPMS entry is confirmed**
— the trigger is whether the `apm` binary or its generated output is bundled into a
customer deliverable. For a binding use-scope ruling, contact **SAP_CIS_ATQ@sap.com**
before the engagement starts.

**What belongs in the project `CLAUDE.md`:**

| Section | Content | Token impact |
|---|---|---|
| Project overview | One paragraph: what this engagement delivers, what it is not | Prevents scope hallucination; saves correction loops |
| Stack and Joule Studio conventions | Framework versions, n8n node conventions, forbidden patterns | Prevents non-compliant code generation |
| Pointers to ADRs and specs | Links to the decision records that govern this project | Replaces inline context with targeted reads |
| Forbidden operations | "Never commit credentials"; "always route write-back through Workflow A" | Eliminates unsafe completions early |

Context management is one of the most important practices in agentic engineering —
ranked above all other practices by architects who have led full agentic CIS projects.
Loading only what the current phase needs is the practical implementation of L1 and L5.

### Top Four Quick Wins from Validated Evidence

The following four techniques have the strongest evidence base from SAP-internal
measurements and research, and apply directly to FDE sprint workflows:

#### 1. Prompt Caching (L3) — Measured 84% cost reduction

**Evidence**: IBD (Intelligent Business Documentation) project measured $25.99 →
$4.22 per session (84% cost reduction) by placing stable system prompt content at
the top of every `sapOrchestration` call and setting the Anthropic cache read rate
(0.1× base input price) for those tokens.

**FDE application**: the agent's system prompt, domain context, and tool descriptions
change rarely within a sprint phase. Place static content first; dynamic per-invocation
context last. Choose cache TTL based on the actual inter-turn gap for this workflow —
the default 1-hour TTL is wrong for many FDE workflows, where sessions may run for days.

**Where it applies**: any `sapOrchestration` node with a substantial system prompt.
Especially high-value for agents that share a common domain knowledge base across many
invocations.

#### 2. MCP & Tool Gating (L1) — ~10K tokens saved per turn

**Evidence**: registering only solution-type-relevant MCP tools (instead of all
available tools) saves approximately 10,000 tokens per agent turn, and 500K+ tokens
over a 50+ turn session. Deterministic routing by solution type selects the correct
tool subset without dynamic selection overhead.

**FDE application**: every Joule Studio session should register only the SAP systems
and MCP tools relevant to this engagement's use case. For an asset management agent,
do not register procurement or HR MCP servers. Establish a routing key (solution type)
that determines which tool subset loads — same key used in skill routing (see below).

**Caution**: do not apply this to supervisor workflows that need cross-domain visibility.
The efficiency gain from narrow tool sets applies to specialist worker agents, not to
generic monitoring or support relay workflows.

#### 3. Session Handoff at Phase Boundaries (L5) — Compounds with caching

**Why it works**: starting a fresh session at each sprint phase boundary (Discovery →
Develop (formerly Execute) → Productization, or within Develop at major artifact milestones) eliminates
re-sent upstream transcript from the previous phase. It also enables per-phase model
routing — a simpler phase can use a lighter model without invalidating the cache from
a heavier reasoning phase.

**The context rot risk**: model performance degrades as input length increases, often
before the nominal context limit is reached. Research shows 11 of 13 models showed
degraded performance at 32K tokens despite a 128K claimed context window. The practical
quality ceiling is approximately 50% of the nominal context window.

**FDE application**: plan explicit session boundaries as part of the sprint workflow
design. At each boundary, write a structured handoff note covering: decisions taken,
artifacts produced, open items, and the entry point for the next session. This is the
FDE-specific application of proactive session compaction (L5).

**Spec-driven development** (the multi-phase pipeline: requirements → architecture →
module specs → implementation plan → code generation, each in its own session) is the
architectural expression of this principle at the engagement level. Each phase produces
an artifact that feeds the next, rather than growing a single session to unmanageable
size.

#### 4. Skill Loading and Deterministic Routing (L1 + L5)

**Why it matters**: without deterministic skill routing, the agent selects skills via
LLM reasoning. In production, this leads to mis-selection — the wrong skill is invoked,
which typically costs 10,000+ tokens before the error is detected and corrected.

**FDE application**: define a routing key (e.g., solution type: asset management,
procurement, manufacturing) at the start of each session. Use this key to load only
the skills relevant to this engagement type. Core skills (project conventions, safety
rules) are always loaded; domain-specific skills are routed by key. This is the same
routing key used in MCP gating — one key, two uses.

**Pairs with APM**: the `apm.yml` per-project setup can encode which skills are loaded
for each solution type, making routing deterministic by default rather than requiring
per-session configuration.

### Input Compression: LLMLingua and Caveman

Two validated input compression techniques are available for reducing input token costs on non-cached, non-structured portions of prompts. Both have been benchmarked in a SAP-internal study (ADR 011, 30 enterprise cases).

| Technique | Input reduction | Quality score | Numeric recall | Latency | Caching-compatible? |
|---|---|---|---|---|---|
| LLMLingua (50% retention) | ~50% | 0.989 | **0.914 (9% drop)** | Higher | **No** |
| LLMLingua (25% retention) | ~75% | 0.882 (**FAIL**) | — | Higher | No |
| Caveman | ~29% | 1.000 (lossless) | 1.000 | < 1 second | Yes |

**Operating guidance:**

- Use **Caveman** as the default: lossless, fast, and caching-compatible. 29% input reduction with zero quality degradation.
- Use **LLMLingua at 50% retention** only for non-numerical, non-financial prompts where a 9% numeric recall drop is acceptable — **never** for batch release quantities, pricing, or financial postings.
- **Never use LLMLingua at 25% retention** — quality falls below threshold (0.882).
- **Critical constraint — caching incompatibility**: input compression defeats prompt caching if applied to cached prompt segments. Compressed tokens are unique per invocation; the cache cannot match them. Only compress the non-cached portion of each prompt. Verify your prompt structure before adding compression — applying compression to the stable, cache-intended portion of a prompt eliminates the caching benefit entirely (see Quick Win 1: Prompt Caching above).

Caveman is preferred for most FDE use cases precisely because its lossless compression preserves cacheability, whereas LLMLingua requires careful partitioning between cached and non-cached prompt segments.

**Cross-reference — Chapter 4 (Precision Guardrails)**: for prompts that handle financial or numerical outputs, LLMLingua at 50% retention is explicitly not safe — a 9% numeric recall drop is unacceptable for batch release quantities, hedge ratios, or pricing figures. Use Caveman (lossless) for all prompts in the precision guardrail domain.

Both ADR 011 (compression) and ADR 014 (startup context, below) are in Draft/In-Review status; treat as recommended practices rather than mandated standards until formal adoption.

### Agent Startup Context — AGENTS.md Pattern

Providing a structured startup context file (`AGENTS.md` or `CLAUDE.md`) at the project root — rather than relying on the agent to explore the codebase or project structure at session start — reduces token consumption per session. An external Codex benchmark (ADR 014, 124 paired tasks) measured the following:

| Metric | Change with AGENTS.md |
|---|---|
| Median output tokens per session | **−16.58%** |
| Median session runtime | **−28.64%** |
| Total tokens (input + output) | +1.29% (more upfront context) |

The trade-off is deliberate: spend tokens once on startup context to avoid repeated exploratory tool calls throughout the session. For FDE projects with established patterns (known API structure, known data model, known workflow conventions), an `AGENTS.md` that documents project structure, key paths, and common patterns will reduce per-session token consumption meaningfully.

The runtime improvement (28.64% lower) is especially significant for FDE sprint work where engineer time — not just token cost — is the constrained resource.

**What to include in a project `AGENTS.md`:** the same content as the `CLAUDE.md` described in §10.3 (Per-Project Agentic Setup): project overview, stack constraints, forbidden operations, and pointers to ADRs and specs. The `AGENTS.md` name is the convention for multi-agent setups (e.g., OpenAI Codex and other agentic frameworks); `CLAUDE.md` is the Claude Code convention. Both patterns apply the same principle and produce the same efficiency gains.

### Measurement During the Sprint: Project-Level Cost Tracking

Token efficiency during the sprint should be measured at the project level, not the
individual level. The HAI proxy (Hyperspace LLM Proxy) tracks consumption per individual
user via SSO identity — there is no built-in project grouping today. The three options
for bridging this gap are analysed below, based on a review of the HAI proxy
architecture and documentation.

**Background: what the HAI proxy does and does not expose**

The proxy captures per-request data at its backend: user identity (SSO/OIDC), model,
token counts (input / output / cache read / cache write), timestamp, and EUR cost. This
data is surfaced through the Desktop App and CLI TUI — individual daily and monthly
totals only. There is no programmatic API to query spending data, no project identifier
attached to requests, and no team or workspace grouping in the current spending layer.
Shared proxies and shared API keys are explicitly prohibited (each engineer must
authenticate individually via SSO). The proxy roadmap does include "workspace-level
dashboards and dedicated rate limits" as a future feature, but no delivery date is set.

**Option A — Project-level API key (shared key per engagement)**

In principle: one API key per FDE engagement, used by all squad members, so costs
accumulate under a single identity. In practice: the local UUID API key is not the
binding — the SSO/OIDC token is. Every request is attributed to the authenticated user,
not to the API key. Shared proxies are explicitly prohibited. This option would require
a backend auth architecture change (service account / non-personal identity per project)
— not a UI extension. Not feasible without significant HAI proxy backend work.

**Option B — Project-level tracking inside the HAI proxy backend**

Adding project/team entities, request-to-project attribution, aggregation queries,
and a management API to the proxy backend. This is structurally what the planned
workspace features describe — the roadmap already points in this direction. The correct
CIS action is to **feed project-level attribution as a formal requirement into the HAI
proxy team's workspace roadmap**, since FDE is a primary driver of the business case.
Not feasible near-term, but the right structural solution.

**Option C — Local accounting sidecar (recommended near-term approach)**

A lightweight tool that sits between AI coding tools and the HAI proxy, intercepts
responses, extracts token counts from the standard API response body, and logs them
with project context:

```
AI Tool (Claude Code)
  → localhost:6656  [local project accounting sidecar]
    → localhost:6655  [HAI proxy — unchanged]
      → Hyperspace backend
```

The Anthropic API response always includes `usage.input_tokens`, `usage.output_tokens`,
`usage.cache_read_input_tokens`, and `usage.cache_creation_input_tokens` in the JSON
body — this is stable and documented. The sidecar forwards all traffic unchanged, reads
token counts from each response, and appends a log line: `timestamp | project_id |
model | tokens | EUR_estimate`. No changes to the HAI proxy; no central database required.

**Project ID auto-detection**: the sidecar reads the project identifier from the
engagement's git repository root (e.g., a `.hai-project` file or the existing `CLAUDE.md`
project name). When Claude Code is invoked from within the engagement repo, the project
context is resolved automatically — the engineer does not need to set it manually.

> **Dependency**: auto-detection assumes a **standard CIS FDE project scaffold** — a
> conventional repo root carrying a `.hai-project` marker or a `CLAUDE.md` with the
> project name. That scaffold does not yet exist as a CIS standard (see
> `context-layer-overview.md`). Until it does, the **manual fallback** is for each
> engineer to set the project ID explicitly in the sidecar config once per engagement.

**Key risk**: engineers must configure their AI tools to point at the sidecar port
(`6656`) instead of the HAI proxy port (`6655`). If this setup is part of the `apm install`
step (see APM section above), it becomes a one-time per-engagement configuration
rather than an ongoing discipline requirement. Data accumulates locally and can be
committed to the engagement repo or posted to a lightweight shared endpoint for
aggregation across the squad.

**Recommendation**: implement Option C as a CIS-internal tooling initiative; feed
Option B into the HAI proxy workspace roadmap as a formal FDE requirement. See
[open-questions.md Q16](open-questions-3a2265d.md) for status tracking.

Until a sidecar or workspace feature is available, the manual fallback is: each
engineer records their HAI proxy monthly total at sprint start and sprint end, reports
the delta to a shared engagement log. Low precision, but sufficient to establish the
first baseline.

**Project LightSpeed and the Hyperspace proxy**: Project LightSpeed uses Claude Code via the Hyperspace proxy — no separate API keys needed. This confirms the Hyperspace proxy as the standard access mechanism for CIS-internal tool development. See `reference_hai_proxy.md` in the CIS memory files for the platform request process and Frontrunners DL contacts.

---

## 10.4 Runtime Efficiency: Deployed Agents

### Different Measurement, Different Levers

Once an Industry AI agent is in production in a customer's BAIP tenant, the token
efficiency measurement shifts entirely to the BAIP-native observability stack:

- **AI Units** (commercial): the per-Action billing model visible in the customer's
  SAP subscription. The contractual metering unit is **0.02 AI Units per Agent Action**
  (1 Agent Action = 1 LLM call + 1 simple tool invocation). Chapter 8 section 8.4
  covers the three-tier step model (Basic / Standard / Advanced at 0.005 / 0.010 /
  0.025 AI Units per step) for detailed cost modelling; the 0.02/Action figure is the
  simplified contractual unit used for floor pricing submissions.
- **OpenTelemetry traces** (operational): per-invocation token counts from the
  generative AI hub, already collected by the BAIP harness automatically. Chapter 8
  sections 8.4–8.5 define the monitoring framework built on top of this signal.

The Hyperspace LLM Proxy is not available here. Any runtime efficiency work must use
these two mechanisms.

**Three runtime billing dimensions**: runtime costs for a deployed FDE agent span three
separate billing pools that map to different contract line items and must be tracked
separately for value case reporting:

| Billing pool | What it covers | Billing unit |
|---|---|---|
| **AI Units** | Agent Actions (LLM calls + tool invocations) | 0.02 AI Units/Action; visible in customer's SAP AI subscription |
| **BTPEA credits** | BTP Enterprise Agreement credits for AppFND runtime infrastructure on BTP Kyma | Separate from AI Units; charged against the customer's BTP contract |
| **BDC capacity units** | Business Data Cloud capacity consumed when agents read from BDC data products | Charged against the customer's BDC subscription |

These three dimensions cannot be consolidated into a single AI Units figure. When
building the FDE value case, confirm which pools the customer has contracted capacity
for — availability of BTPEA credits and BDC capacity units affects deployment
feasibility independently of AI Units availability.

For **Topology A deployments** (customer-managed tenant), access to runtime token data
requires customer consent — the same consent required for operational monitoring
generally. This is the Topology A data access constraint from Chapter 6 section 6.4.
Runtime efficiency optimization for Topology A agents cannot proceed without this
consent in place.

### L0 at Agent Design Time

The most impactful runtime efficiency decision is made at design time: does this step
need LLM reasoning, or can a deterministic Code node do it?

In Joule Studio (n8n), every step that passes through `sapOrchestration` or the
AI Agent node costs tokens. Every step that runs in a Code node costs compute — but
not LLM tokens. The design rule is:

> **Use a Code node for any step whose logic can be expressed as deterministic rules.
> Use `sapOrchestration` only when the step genuinely requires language understanding,
> reasoning across ambiguous inputs, or natural language generation.**

Common cases where FDE teams over-use LLM nodes:

| Step | Often done with LLM | Better approach |
|---|---|---|
| Format conversion (JSON → table, XML → JSON) | `sapOrchestration` | Code node |
| Threshold comparison (value > threshold?) | `sapOrchestration` | Code node (If node) |
| Field extraction from structured output | `sapOrchestration` | Code node |
| PII detection on known-structure fields | `sapOrchestration` | `sapDPI` with explicit config |
| Routing logic (which workflow path?) | AI Agent tool selection | Switch node with deterministic routing key |

The supervisor workflows are a positive example: Workflow A's range-check and unit-check
rules run in a Code node. The LLM critic in Workflow A is invoked only when the
deterministic Code node rules cannot make a definitive verdict — this is L0 applied at
workflow design level.

### Joule Studio Design Choices for Runtime Efficiency

Beyond L0, several design choices at workflow build time directly affect runtime token
consumption:

**Output format discipline (L2)**: configure `sapOrchestration`'s `outputSchema` to
return compact, structured JSON rather than verbose natural language. Compact structured
JSON is significantly cheaper than equivalent YAML or verbose JSON. Set `max_tokens`
in the output schema to prevent verbosity drift — an output ceiling that reflects the
actual maximum expected output size, not the model's default limit.

**MCP call batching (L1)**: where the workflow makes multiple reads from the same SAP
system in one turn, batch them via the `sapMcpClient` batch call pattern rather than
sequential single-call reads. Each separate MCP tool call carries overhead (tool
description retransmission, result formatting); batching reduces this overhead.

**Context scoping per phase (L1 + L5)**: agent workflows that accumulate conversation
history across many turns within a session should apply proactive compaction before the
context degrades. For long-running agent sessions in production (e.g., an asset
management agent processing an extended maintenance event), the session compaction
trigger at 50% of nominal context window applies to runtime as much as to build-time.

### The Supervisor as Efficiency Gate

The supervisor pattern (Chapter 2) has a token efficiency benefit that is not its
primary purpose but is a practical gain: the supervisor intercepts output before
write-back. If an agent has produced a mis-formed or off-range output after excessive
token expenditure, the supervisor rejects it before it triggers a human approval flow
— containing the cost of the failure to a single invocation rather than letting the
error propagate into the human loop.

Supervisor Workflow B (Telemetry Monitor, Chapter 8 section 8.5) already monitors
token anomalies as part of its scheduled monitoring cycle. The specific signals —
per-invocation tokens vs. rolling baseline, runaway detection at 5× p99, daily budget
threshold — are defined there. The supervisor design and the efficiency monitoring
design are the same workflow.

---

## 10.5 The Quality–Efficiency Connection

### Token Cost as a Measurable Sub-Dimension of Tool Call Efficiency

Chapter 1 (section 1.2, Dimension 1) lists tool call efficiency as a technical quality
metric: "Did the agent reach the goal with the minimum necessary tool calls?" Token
cost makes this measurable in monetary terms.

An agent that reaches the same business outcome with half the tool calls, half the
context re-sent, and half the output verbosity is better — not just cheaper. It is
also faster, less likely to hit context limits, and more predictable. Token efficiency
is not in tension with quality; at the operational level, high token consumption is
frequently a quality symptom:

| Token anomaly signal | Likely quality root cause |
|---|---|
| Input token spike (consistent, not runaway) | Context expansion creep — growing grounding data, accumulated history |
| Runaway token spike (single invocation) | Stuck retry loop or infinite tool call cycle |
| High cost per committed output | Many failed invocations before successful write-back; high human override rate driving rework |
| Gradually increasing cost per invocation | Prompt drift — system prompt growing over time without audit |
| High cost per escalation | Agent consuming tokens before recognizing it cannot complete the task |

These signals are already detectable via Workflow B (Chapter 8 section 8.5). Framing
them as efficiency signals — not just cost signals — connects the telemetry monitoring
to the quality framework.

### Goodhart's Law Warning

Chapter 1 warns that optimizing for any single metric produces misleading results
(Goodhart's Law). This applies directly to token efficiency: minimizing token count
while ignoring task completion quality is the wrong optimization target.

An agent that reaches correct outcomes with fewer tokens is genuinely better. An agent
that reaches fewer correct outcomes but uses fewer tokens is worse, regardless of what
the cost dashboard shows.

The correct efficiency metric — from SAP's L6 governance level — is **cost per correct
outcome**: the total LLM cost divided by the number of outputs that were correct,
approved, and committed to the system of record. This is the runtime counterpart of
the value-based contract logic: just as the contract is priced against customer benefit,
efficiency is measured against delivered value, not raw token throughput.

For FDE engagements, "cost per correct outcome" can be approximated as:

```
Cost per correct outcome = Total AI Units consumed
                           ────────────────────────────────────────────
                           Outputs committed to system of record (Dimension 2 process KPI)
```

This metric should be included in the FDE value case alongside the business outcome
metrics (Chapter 1, Dimension 3).

At **build-time** the same L6 logic applies with a different numerator and denominator —
**Hyperspace EUR spent / number of correct (accepted, merged) build artifacts** (e.g.
EUR per accepted spec, workflow, or agent build):

```
Cost per correct outcome (build-time) = Total Hyperspace EUR spent
                                        ──────────────────────────────────────────
                                        Correct build artifacts (accepted / merged)
```

This build-time variant is what justifies the ✓ build-time marking for the metric in
the §10.7 quick-wins table; the AI-Units formula above is the runtime variant.

---

## 10.6 Known Gaps and Open Questions

The following gaps limit the current ability to implement full token efficiency
measurement and governance for CIS engagements.

### Q16 — Hyperspace LLM Proxy per-project cost tracking

**Status**: Open — needs verification with the Hyperspace/DTS tooling team

Can Hyperspace LLM Proxy costs be tracked and reported at the project or FDE engagement
level, rather than only at the individual developer level? Without project-level
grouping, the build-time cost pool cannot be managed as an engagement expense — only
as individual consumption. Per-engagement baseline measurement (the foundation for
any efficiency comparison across engagements) requires this capability.

**Reference**: [Chapter 10, section 10.3](token-efficiency-f8f90a7.md) — build-time
measurement; see also [open-questions.md Q16](open-questions-3a2265d.md)

### Q17 — AI Units vs. Hyperspace: relationship and runtime proxy availability

**Status**: Open — needs confirmation with the BAIP commercial and tooling teams

Are AI Units (BAIP runtime billing) and Hyperspace (development tool billing)
entirely separate cost streams with no shared tracking or reporting? Is there any
proxy equivalent to Hyperspace available for monitoring deployed agent token costs
from a SAP operator perspective — outside of the customer-visible AI Units billing?

**Reference**: [Chapter 8, section 8.4](production-operations-324bb02.md) —
AI Units billing model; see also [open-questions.md Q17](open-questions-3a2265d.md)

### No engagement baseline measurement

No FDE engagement has yet measured build-time token efficiency systematically. The
84% caching reduction from the IBD project provides a reference point for a specific
technique; what is missing is a baseline measurement approach that engagements could
apply consistently from the start of Develop, enabling comparison across engagements
over time. Establishing this baseline — even informally in the next two or three
engagements — should be a CIS T&O priority.

### Runtime monitoring in Topology A requires customer consent

As noted in section 10.4 above and in Chapter 6 section 6.4, access to per-invocation
runtime telemetry in Topology A (customer-managed tenant) requires the customer's
explicit consent. The runtime efficiency design in this chapter is complete — but
it is not deployable in Topology A engagements until the consent question in Discovery
has been resolved and the legal basis established.

---

## 10.7 Recommended Quick-Wins Table

The following table summarizes the techniques with the strongest evidence-to-effort
ratio, organized for FDE teams starting a new engagement.

| Technique | L-level | Evidence | Effort | Build-time | Runtime | Where in concept |
|---|---|---|---|---|---|---|
| Prompt caching (stable content first; calibrate TTL) | L3 | Measured: 84% cost reduction (IBD project) | Low — design convention in sapOrchestration | ✓ | ✓ | Ch10 §10.3; confirm TTL with Ch8 §8.4 baseline |
| MCP / tool gating by solution type | L1 | Estimated: ~10K tokens/turn saved | Low — APM setup | ✓ | ✓ | Ch10 §10.3; Ch5 §5.4 worker agent pattern |
| Session handoff notes at phase boundaries | L5 | Research-backed (context rot; 11/13 models degraded at 32K) | Low — process convention | ✓ | (partial) | Ch10 §10.3; DTS spec-driven development pattern |
| Code node for deterministic steps (L0 applied) | L0 | Design principle — no token cost for deterministic logic | Low-Medium — requires conscious workflow review | — | ✓ | Ch10 §10.4; Ch5 §5.4 (already embedded in supervisor pattern) |
| Compact JSON output format in sapOrchestration | L2 | Engineering principle | Low — configuration choice | ✓ | ✓ | Ch10 §10.4; Ch5 §5.4 |
| Skill / agent routing by deterministic key | L1 + L5 | Estimated: eliminates 10K+ tokens per mis-selection | Low — APM + CLAUDE.md setup | ✓ | ✓ | Ch10 §10.3 |
| Per-project APM setup (CIS Starter Kit apm.yml) | L1 | Structural — prevents anti-pattern accumulation | Medium — initial setup; low thereafter | ✓ | — | Ch10 §10.3; Ch9 §9.4 |
| Cost per correct outcome metric | L6 | Aligns with value-based contract logic | Medium — requires measurement instrumentation | ✓ | ✓ | Ch10 §10.5; Ch1 §1.2 |

**Recommendation for a new FDE engagement starting Develop**: implement the first
three rows in week 0 of Develop as part of the agentic project setup — they require
no special tooling, only conscious configuration choices. Add the fourth and fifth
rows during workflow design review. Track the cost per correct outcome metric from
the first EAC deployment.
