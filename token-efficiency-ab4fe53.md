<!-- loioab4fe535cf8743a8997cff355fce1997 -->

# Token Efficiency in Industry AI Delivery

Token cost in Industry AI delivery splits into two separate cost pools — build-time and runtime — with different measurement mechanisms, owners, and optimization levers. This topic defines both pools, applies SAP's L0–L6 Token Optimization Framework to each, and provides evidence-based quick wins for FDE teams.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_why_efficiency_matters"/>

## Why Token Efficiency Matters

### The Shift from Subsidy to Accountability

The economics of AI development have changed. SAP CTO Philipp Herzig has described the shift as moving from a "token-subsidy era" — where LLM usage costs were absorbed without visibility — to a "token-scarcity era" where accuracy, price, and performance must be optimized together. SAP's Token Consumption Governance Framework, which went live on 17 August 2026 via the Hyperspace LLM Proxy, made this shift concrete for SAP-internal tooling. Token budgets and consumption plans now apply to developer tools (Claude Code, Codex, Pi, opencode) at the individual and team level.

This is not a signal to use less AI — it is a signal that token consumption must be deliberate and measurable. For CIS and FDE teams, deliberate means: knowing where tokens are spent, having techniques to reduce waste, and framing efficiency not as token count but as value delivered per token spent.

### Two Cost Pools with Different Owners

The central insight is that token cost in Industry AI delivery splits into two entirely separate cost pools. They have different measurement mechanisms, different owners, and different optimization levers.

<table>
<caption>Token Cost Pools</caption>
<tr>
<th valign="top">Cost Pool</th>
<th valign="top">What It Covers</th>
<th valign="top">How It Is Measured</th>
<th valign="top">Who Owns It</th>
</tr>
<tr>
<td valign="top">

**Build-time**

</td>
<td valign="top">

FDE engineers using Claude Code and AI coding tools during the sprint

</td>
<td valign="top">

Hyperspace LLM Proxy — SAP-internal only

</td>
<td valign="top">

CIS / FDE team

</td>
</tr>
<tr>
<td valign="top">

**Runtime**

</td>
<td valign="top">

Deployed Industry AI agents processing business transactions for the customer

</td>
<td valign="top">

BAIP AI Units + OpenTelemetry (built into BAIP harness)

</td>
<td valign="top">

Customer (billing) + CIS (operational monitoring)

</td>
</tr>
</table>

> ### Note:  
> The runtime pool is itself multi-dimensional. The runtime pool decomposes at deployment into three separate, non-consolidatable billing pools — AI Units + BTPEA credits + BDC capacity units — that map to different contract line items. See the Runtime Efficiency section for the breakdown and its value-case implications.

**Critical constraint**: the Hyperspace LLM Proxy is an SAP-internal tool. It is not available for deployed agents running in a customer's BAIP tenant at runtime. Any efficiency measurement of production agents relies exclusively on the BAIP-native observability stack: AI Units for commercial billing and OpenTelemetry traces for operational monitoring (see [Production Operations](production-operations-79a39ed.md) for the operational design).

CIS carries accountability for both pools: build-time efficiency protects the sprint budget; runtime efficiency protects the customer's value case.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_l0_l6_framework"/>

## SAP's L0–L6 Token Optimization Framework

SAP's internal Token Optimization Cheat Sheet defines seven levels of optimization, numbered L0–L6. The framework progresses from the cheapest and most impactful interventions (L0) to governance-layer instrumentation (L6).

<table>
<caption>L0–L6 Token Optimization Framework</caption>
<tr>
<th valign="top">Level</th>
<th valign="top">Name</th>
<th valign="top">Core Action</th>
</tr>
<tr>
<td valign="top">

L0

</td>
<td valign="top">

**ELIMINATE**

</td>
<td valign="top">

Use deterministic tools first; ask whether an LLM is necessary at all for this step

</td>
</tr>
<tr>
<td valign="top">

L1

</td>
<td valign="top">

**INPUT CONTROL**

</td>
<td valign="top">

Lean prompts; only solution-relevant MCP tools registered; only the context the agent needs for this phase

</td>
</tr>
<tr>
<td valign="top">

L2

</td>
<td valign="top">

**OUTPUT CONTROL**

</td>
<td valign="top">

Return diffs not rewrites; set response ceilings; suppress chain-of-thought reasoning where the trace is not needed

</td>
</tr>
<tr>
<td valign="top">

L3

</td>
<td valign="top">

**CACHING**

</td>
<td valign="top">

Put stable, rarely-changing content at the beginning of prompts to exploit cache reads; stable `CLAUDE.md` for build-time sessions

</td>
</tr>
<tr>
<td valign="top">

L4

</td>
<td valign="top">

**MODEL ROUTING**

</td>
<td valign="top">

Route by complexity: lightweight model for simple tasks, frontier model for hard reasoning; batch and async for non-urgent work

</td>
</tr>
<tr>
<td valign="top">

L5

</td>
<td valign="top">

**AGENT CAPS**

</td>
<td valign="top">

Manage session length; use session compaction; write handoff notes at phase boundaries

</td>
</tr>
<tr>
<td valign="top">

L6

</td>
<td valign="top">

**GOVERNANCE**

</td>
<td valign="top">

Track cost per correct outcome, not per token; establish measurement baselines; set accountability structures

</td>
</tr>
</table>

The framework applies to both cost pools but with different emphasis:

-   For **build-time** (FDE engineers), L0–L5 are practical daily habits and tool configuration choices. L6 is the engagement-level measurement question.
-   For **runtime** (deployed agents), L0 is the most consequential — agent workflow design determines which steps require LLM reasoning and which can use deterministic Code nodes. L1–L3 are encoded into the Joule Studio workflow design and the `sapOrchestration` configuration.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_build_time_efficiency"/>

## Build-Time Efficiency: FDE Engineer Guidelines

### Current State

No standardized token efficiency guidelines currently exist for FDE teams. The SAP Token Consumption Governance Framework (L6) is now in place for individual developer tooling via Hyperspace, but CIS does not yet have:

-   Per-engagement or per-project cost tracking
-   A prescribed agentic project setup that embeds L0–L5 practices from day one
-   A baseline measurement approach that would let engagements compare efficiency over time

### Per-Project Agentic Setup (L1 + L5)

The single highest-impact build-time change is establishing a per-project, reproducible agentic configuration rather than relying on individual engineers' global tool installations.

**The anti-pattern**: each engineer installs whatever MCP servers and skills they prefer globally. Every session starts with a different tool set, different context, and no shared project conventions. Token waste is structurally guaranteed: the wrong tools are registered, project context must be re-established from scratch, and there is no common baseline to measure against.

**The better pattern**: a per-project `apm.yml` (Agent Package Manager, a Microsoft OSS tool for managing agentic project configurations) that declares:

-   Which MCP servers are registered for this project type (not all MCP servers globally)
-   Which skills apply to this engagement's workflow
-   A project `CLAUDE.md` encoding the project overview, stack constraints, forbidden operations, and pointers to ADRs and specs

The `apm install` command deploys the declared tool set; `apm compile` generates the `CLAUDE.md`. This gives every engineer on the FDE squad an identical, version-locked starting point.

> ### Note:  
> `apm` is a Microsoft-originated, MIT-licensed open-source tool (github.com/microsoft/apm) adopted by SAP — not an SAP-built tool. No confirmed SAP Software Hub / PPMS registration could be found; the governing boundary is build-time-vs-shipped (SDOL-005 / PPMS). Build-time use on SAP-internal CI and developer workstations is acceptable because `apm install` writes config into harness directories and does not appear in the deployed customer artifact. **Customer-environment deployment is uncleared until a PPMS entry is confirmed.** For a binding use-scope ruling, contact **SAP_CIS_ATQ@sap.com** before the engagement starts.

The [CIS Delivery Efficiency and Reusable Components](reusable-components-62438fe.md) Starter Kit is the natural home for an `apm.yml` template per common engagement type.

**What belongs in the project `CLAUDE.md`:**

<table>
<caption">Project CLAUDE.md Sections</caption>
<tr>
<th valign="top">Section</th>
<th valign="top">Content</th>
<th valign="top">Token Impact</th>
</tr>
<tr>
<td valign="top">

Project overview

</td>
<td valign="top">

One paragraph: what this engagement delivers, what it is not

</td>
<td valign="top">

Prevents scope hallucination; saves correction loops

</td>
</tr>
<tr>
<td valign="top">

Stack and Joule Studio conventions

</td>
<td valign="top">

Framework versions, n8n node conventions, forbidden patterns

</td>
<td valign="top">

Prevents non-compliant code generation

</td>
</tr>
<tr>
<td valign="top">

Pointers to ADRs and specs

</td>
<td valign="top">

Links to the decision records that govern this project

</td>
<td valign="top">

Replaces inline context with targeted reads

</td>
</tr>
<tr>
<td valign="top">

Forbidden operations

</td>
<td valign="top">

"Never commit credentials"; "always route write-back through Workflow A"

</td>
<td valign="top">

Eliminates unsafe completions early

</td>
</tr>
</table>

Context management is one of the most important practices in agentic engineering. Loading only what the current phase needs is the practical implementation of L1 and L5.

### Top Four Quick Wins from Validated Evidence

#### 1. Prompt Caching (L3) — Measured 84% Cost Reduction

**Evidence**: IBD (Intelligent Business Documentation) project measured $25.99 → $4.22 per session (84% cost reduction) by placing stable system prompt content at the top of every `sapOrchestration` call and setting the Anthropic cache read rate (0.1× base input price) for those tokens.

**FDE application**: the agent's system prompt, domain context, and tool descriptions change rarely within a sprint phase. Place static content first; dynamic per-invocation context last. Choose cache TTL based on the actual inter-turn gap for this workflow — the default 1-hour TTL is wrong for many FDE workflows, where sessions may run for days.

#### 2. MCP and Tool Gating (L1) — ~10K Tokens Saved Per Turn

**Evidence**: registering only solution-type-relevant MCP tools (instead of all available tools) saves approximately 10,000 tokens per agent turn, and 500K+ tokens over a 50+ turn session. Deterministic routing by solution type selects the correct tool subset without dynamic selection overhead.

**FDE application**: every Joule Studio session should register only the SAP systems and MCP tools relevant to this engagement's use case. Do not apply this to supervisor workflows that need cross-domain visibility — the efficiency gain from narrow tool sets applies to specialist worker agents, not to generic monitoring or support relay workflows.

#### 3. Session Handoff at Phase Boundaries (L5) — Compounds with Caching

**Why it works**: starting a fresh session at each sprint phase boundary (Discovery → Develop → Productization, or within Develop at major artifact milestones) eliminates re-sent upstream transcript from the previous phase. It also enables per-phase model routing — a simpler phase can use a lighter model without invalidating the cache from a heavier reasoning phase.

**The context rot risk**: model performance degrades as input length increases, often before the nominal context limit is reached. Research shows 11 of 13 models showed degraded performance at 32K tokens despite a 128K claimed context window. The practical quality ceiling is approximately 50% of the nominal context window.

**FDE application**: plan explicit session boundaries as part of the sprint workflow design. At each boundary, write a structured handoff note covering: decisions taken, artifacts produced, open items, and the entry point for the next session.

Spec-driven development — the multi-phase pipeline: requirements → architecture → module specs → implementation plan → code generation, each in its own session — is the architectural expression of this principle at the engagement level.

#### 4. Skill Loading and Deterministic Routing (L1 + L5)

**Why it matters**: without deterministic skill routing, the agent selects skills via LLM reasoning. In production, this leads to mis-selection — the wrong skill is invoked, which typically costs 10,000+ tokens before the error is detected and corrected.

**FDE application**: define a routing key (for example, solution type: asset management, procurement, manufacturing) at the start of each session. Use this key to load only the skills relevant to this engagement type. Core skills (project conventions, safety rules) are always loaded; domain-specific skills are routed by key.

### Input Compression: LLMLingua and Caveman

Two validated input compression techniques are available for reducing input token costs on non-cached, non-structured portions of prompts. Both have been benchmarked in a SAP-internal study (ADR 011, 30 enterprise cases).

<table>
<caption>Input Compression Technique Comparison</caption>
<tr>
<th valign="top">Technique</th>
<th valign="top">Input Reduction</th>
<th valign="top">Quality Score</th>
<th valign="top">Numeric Recall</th>
<th valign="top">Latency</th>
<th valign="top">Caching-Compatible?</th>
</tr>
<tr>
<td valign="top">

LLMLingua (50% retention)

</td>
<td valign="top">

~50%

</td>
<td valign="top">

0.989

</td>
<td valign="top">

**0.914 (9% drop)**

</td>
<td valign="top">

Higher

</td>
<td valign="top">

**No**

</td>
</tr>
<tr>
<td valign="top">

LLMLingua (25% retention)

</td>
<td valign="top">

~75%

</td>
<td valign="top">

0.882 (**FAIL**)

</td>
<td valign="top">

—

</td>
<td valign="top">

Higher

</td>
<td valign="top">

No

</td>
</tr>
<tr>
<td valign="top">

Caveman

</td>
<td valign="top">

~29%

</td>
<td valign="top">

1.000 (lossless)

</td>
<td valign="top">

1.000

</td>
<td valign="top">

&lt; 1 second

</td>
<td valign="top">

Yes

</td>
</tr>
</table>

**Operating guidance:**

-   Use **Caveman** as the default: lossless, fast, and caching-compatible. 29% input reduction with zero quality degradation.
-   Use **LLMLingua at 50% retention** only for non-numerical, non-financial prompts where a 9% numeric recall drop is acceptable — **never** for batch release quantities, pricing, or financial postings.
-   **Never use LLMLingua at 25% retention** — quality falls below threshold (0.882).
-   **Critical constraint — caching incompatibility**: input compression defeats prompt caching if applied to cached prompt segments. Only compress the non-cached portion of each prompt.

Cross-reference — [Precision Guardrails](precision-guardrails-331971d.md): for prompts that handle financial or numerical outputs, LLMLingua at 50% retention is explicitly not safe — a 9% numeric recall drop is unacceptable for batch release quantities, hedge ratios, or pricing figures.

### Agent Startup Context — AGENTS.md Pattern

Providing a structured startup context file (`AGENTS.md` or `CLAUDE.md`) at the project root — rather than relying on the agent to explore the codebase at session start — reduces token consumption per session. An external Codex benchmark (ADR 014, 124 paired tasks) measured:

<table>
<caption>AGENTS.md Impact Metrics</caption>
<tr>
<th valign="top">Metric</th>
<th valign="top">Change with AGENTS.md</th>
</tr>
<tr>
<td valign="top">

Median output tokens per session

</td>
<td valign="top">

**−16.58%**

</td>
</tr>
<tr>
<td valign="top">

Median session runtime

</td>
<td valign="top">

**−28.64%**

</td>
</tr>
<tr>
<td valign="top">

Total tokens (input + output)

</td>
<td valign="top">

+1.29% (more upfront context)

</td>
</tr>
</table>

The trade-off is deliberate: spend tokens once on startup context to avoid repeated exploratory tool calls throughout the session. The runtime improvement (28.64% lower) is especially significant for FDE sprint work where engineer time — not just token cost — is the constrained resource.

The `AGENTS.md` name is the convention for multi-agent setups; `CLAUDE.md` is the Claude Code convention. Both patterns apply the same principle and produce the same efficiency gains.

### Measurement During the Sprint: Project-Level Cost Tracking

Token efficiency during the sprint should be measured at the project level, not the individual level. The HAI proxy (Hyperspace LLM Proxy) tracks consumption per individual user via SSO identity — there is no built-in project grouping today.

**Background**: the proxy captures per-request data at its backend: user identity (SSO/OIDC), model, token counts, timestamp, and EUR cost. There is no programmatic API to query spending data, no project identifier attached to requests, and no team or workspace grouping in the current spending layer. Shared proxies and shared API keys are explicitly prohibited.

Three options for bridging this gap:

<table>
<caption>Project-Level Cost Tracking Options</caption>
<tr>
<th valign="top">Option</th>
<th valign="top">Approach</th>
<th valign="top">Feasibility</th>
</tr>
<tr>
<td valign="top">

A — Project API key (shared)

</td>
<td valign="top">

One key per FDE engagement used by all engineers

</td>
<td valign="top">

Not feasible — auth is SSO-bound, not key-bound; shared proxies prohibited

</td>
</tr>
<tr>
<td valign="top">

B — Project tracking in HAI proxy backend

</td>
<td valign="top">

Add project/team entities + attribution + management API to proxy backend

</td>
<td valign="top">

Requires backend work; aligned with planned workspace features; medium-term

</td>
</tr>
<tr>
<td valign="top">

C — Local accounting sidecar (recommended near-term)

</td>
<td valign="top">

Lightweight tool sits between AI coding tools and the HAI proxy, intercepts responses, extracts token counts from the standard API response body, and logs them with project context

</td>
<td valign="top">

Feasible now; project ID auto-detected from repo

</td>
</tr>
</table>

**Option C sidecar architecture:**

```
AI Tool (Claude Code)
  → localhost:6656  [local project accounting sidecar]
    → localhost:6655  [HAI proxy — unchanged]
      → Hyperspace backend
```

The Anthropic API response always includes `usage.input_tokens`, `usage.output_tokens`, `usage.cache_read_input_tokens`, and `usage.cache_creation_input_tokens` in the JSON body. The sidecar forwards all traffic unchanged, reads token counts from each response, and appends a log line: `timestamp | project_id | model | tokens | EUR_estimate`.

**Dependency**: auto-detection assumes a standard CIS FDE project scaffold — a conventional repo root carrying a `.hai-project` marker or a `CLAUDE.md` with the project name. That scaffold does not yet exist as a CIS standard (see [CIS Context Layer Overview](cis-context-layer-overview-bd1e371.md)). Until it does, the manual fallback is for each engineer to set the project ID explicitly in the sidecar config once per engagement.

**Recommendation**: implement Option C as a CIS-internal tooling initiative; feed Option B into the HAI proxy workspace roadmap as a formal FDE requirement. See [Open Questions](open-questions-9d98d9f.md) Q16 for status tracking.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_runtime_efficiency"/>

## Runtime Efficiency: Deployed Agents

### Different Measurement, Different Levers

Once an Industry AI agent is in production in a customer's BAIP tenant, the token efficiency measurement shifts entirely to the BAIP-native observability stack:

-   **AI Units** (commercial): the per-Action billing model visible in the customer's SAP subscription. The contractual metering unit is **0.02 AI Units per Agent Action** (1 Agent Action = 1 LLM call + 1 simple tool invocation). [Production Operations](production-operations-79a39ed.md) covers the three-tier step model (Basic / Standard / Advanced at 0.005 / 0.010 / 0.025 AI Units per step) for detailed cost modelling.
-   **OpenTelemetry traces** (operational): per-invocation token counts from the generative AI hub, already collected by the BAIP harness automatically.

**Three runtime billing dimensions:**

<table>
<caption>Runtime Billing Pools</caption>
<tr>
<th valign="top">Billing Pool</th>
<th valign="top">What It Covers</th>
<th valign="top">Billing Unit</th>
</tr>
<tr>
<td valign="top">

**AI Units**

</td>
<td valign="top">

Agent Actions (LLM calls + tool invocations)

</td>
<td valign="top">

0.02 AI Units/Action; visible in customer's SAP AI subscription

</td>
</tr>
<tr>
<td valign="top">

**BTPEA credits**

</td>
<td valign="top">

BTP Enterprise Agreement credits for AppFND runtime infrastructure on BTP Kyma

</td>
<td valign="top">

Separate from AI Units; charged against the customer's BTP contract

</td>
</tr>
<tr>
<td valign="top">

**BDC capacity units**

</td>
<td valign="top">

Business Data Cloud capacity consumed when agents read from BDC data products

</td>
<td valign="top">

Charged against the customer's BDC subscription

</td>
</tr>
</table>

These three dimensions cannot be consolidated into a single AI Units figure. When building the FDE value case, confirm which pools the customer has contracted capacity for — availability of BTPEA credits and BDC capacity units affects deployment feasibility independently of AI Units availability.

For **Topology A deployments** (customer-managed tenant), access to runtime token data requires customer consent. Runtime efficiency optimization for Topology A agents cannot proceed without this consent in place.

### L0 at Agent Design Time

The most impactful runtime efficiency decision is made at design time: does this step need LLM reasoning, or can a deterministic Code node do it?

In Joule Studio (n8n), every step that passes through `sapOrchestration` or the AI Agent node costs tokens. Every step that runs in a Code node costs compute — but not LLM tokens. The design rule:

> **Use a Code node for any step whose logic can be expressed as deterministic rules. Use `sapOrchestration` only when the step genuinely requires language understanding, reasoning across ambiguous inputs, or natural language generation.**

Common cases where FDE teams over-use LLM nodes:

<table>
<caption>LLM Over-Use Cases and Better Approaches</caption>
<tr>
<th valign="top">Step</th>
<th valign="top">Often Done with LLM</th>
<th valign="top">Better Approach</th>
</tr>
<tr>
<td valign="top">

Format conversion (JSON → table, XML → JSON)

</td>
<td valign="top">

`sapOrchestration`

</td>
<td valign="top">

Code node

</td>
</tr>
<tr>
<td valign="top">

Threshold comparison (value > threshold?)

</td>
<td valign="top">

`sapOrchestration`

</td>
<td valign="top">

Code node (If node)

</td>
</tr>
<tr>
<td valign="top">

Field extraction from structured output

</td>
<td valign="top">

`sapOrchestration`

</td>
<td valign="top">

Code node

</td>
</tr>
<tr>
<td valign="top">

PII detection on known-structure fields

</td>
<td valign="top">

`sapOrchestration`

</td>
<td valign="top">

`sapDPI` with explicit config

</td>
</tr>
<tr>
<td valign="top">

Routing logic (which workflow path?)

</td>
<td valign="top">

AI Agent tool selection

</td>
<td valign="top">

Switch node with deterministic routing key

</td>
</tr>
</table>

The supervisor workflows are a positive example: Workflow A's range-check and unit-check rules run in a Code node. The LLM critic in Workflow A is invoked only when the deterministic Code node rules cannot make a definitive verdict.

### Joule Studio Design Choices for Runtime Efficiency

Beyond L0, several design choices at workflow build time directly affect runtime token consumption:

-   **Output format discipline (L2)**: configure `sapOrchestration`'s `outputSchema` to return compact, structured JSON rather than verbose natural language. Set `max_tokens` to prevent verbosity drift.
-   **MCP call batching (L1)**: where the workflow makes multiple reads from the same SAP system in one turn, batch them via the `sapMcpClient` batch call pattern rather than sequential single-call reads.
-   **Context scoping per phase (L1 + L5)**: agent workflows that accumulate conversation history across many turns should apply proactive compaction before the context degrades. The session compaction trigger at 50% of nominal context window applies to runtime as much as to build-time.

### The Supervisor as Efficiency Gate

The supervisor pattern (see [Supervisor Agent Pattern](supervisor-agent-pattern-fa25f56.md)) has a token efficiency benefit that is not its primary purpose: the supervisor intercepts output before write-back. If an agent has produced a mis-formed or off-range output after excessive token expenditure, the supervisor rejects it before it triggers a human approval flow — containing the cost of the failure to a single invocation rather than letting the error propagate into the human loop.

Supervisor Workflow B ([Production Operations](production-operations-79a39ed.md)) already monitors token anomalies as part of its scheduled monitoring cycle. The specific signals — per-invocation tokens vs. rolling baseline, runaway detection at 5× p99, daily budget threshold — are defined there.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_quality_efficiency"/>

## The Quality–Efficiency Connection

### Token Cost as a Measurable Sub-Dimension of Tool Call Efficiency

[Agent Quality Framework](agent-quality-framework-58e930e.md) lists tool call efficiency as a technical quality metric: "Did the agent reach the goal with the minimum necessary tool calls?" Token cost makes this measurable in monetary terms.

An agent that reaches the same business outcome with half the tool calls, half the context re-sent, and half the output verbosity is better — not just cheaper. It is also faster, less likely to hit context limits, and more predictable. Token efficiency is not in tension with quality; at the operational level, high token consumption is frequently a quality symptom:

<table>
<caption>Token Anomaly Signals and Quality Root Causes</caption>
<tr>
<th valign="top">Token Anomaly Signal</th>
<th valign="top">Likely Quality Root Cause</th>
</tr>
<tr>
<td valign="top">

Input token spike (consistent, not runaway)

</td>
<td valign="top">

Context expansion creep — growing grounding data, accumulated history

</td>
</tr>
<tr>
<td valign="top">

Runaway token spike (single invocation)

</td>
<td valign="top">

Stuck retry loop or infinite tool call cycle

</td>
</tr>
<tr>
<td valign="top">

High cost per committed output

</td>
<td valign="top">

Many failed invocations before successful write-back; high human override rate driving rework

</td>
</tr>
<tr>
<td valign="top">

Gradually increasing cost per invocation

</td>
<td valign="top">

Prompt drift — system prompt growing over time without audit

</td>
</tr>
<tr>
<td valign="top">

High cost per escalation

</td>
<td valign="top">

Agent consuming tokens before recognizing it cannot complete the task

</td>
</tr>
</table>

These signals are already detectable via Workflow B ([Production Operations](production-operations-79a39ed.md)).

### Goodhart's Law Warning

[Agent Quality Framework](agent-quality-framework-58e930e.md) warns that optimizing for any single metric produces misleading results (Goodhart's Law). This applies directly to token efficiency: minimizing token count while ignoring task completion quality is the wrong optimization target.

An agent that reaches correct outcomes with fewer tokens is genuinely better. An agent that reaches fewer correct outcomes but uses fewer tokens is worse, regardless of what the cost dashboard shows.

The correct efficiency metric — from SAP's L6 governance level — is **cost per correct outcome**: the total LLM cost divided by the number of outputs that were correct, approved, and committed to the system of record. For FDE engagements:

```
Cost per correct outcome (runtime) = Total AI Units consumed
                                     ────────────────────────────────────────────
                                     Outputs committed to system of record

Cost per correct outcome (build-time) = Total Hyperspace EUR spent
                                        ──────────────────────────────────────────
                                        Correct build artifacts (accepted / merged)
```

This metric should be included in the FDE value case alongside the business outcome metrics from [Agent Quality Framework](agent-quality-framework-58e930e.md).


<a name="loioab4fe535cf8743a8997cff355fce1997__section_known_gaps"/>

## Known Gaps and Open Questions

### Q16 — Hyperspace LLM Proxy Per-Project Cost Tracking

**Status**: Open — needs verification with the Hyperspace/DTS tooling team

Can Hyperspace LLM Proxy costs be tracked and reported at the project or FDE engagement level, rather than only at the individual developer level? Without project-level grouping, the build-time cost pool cannot be managed as an engagement expense. See [Open Questions](open-questions-9d98d9f.md) Q16 for the full analysis.

### Q17 — AI Units vs. Hyperspace: Relationship and Runtime Proxy Availability

**Status**: Open — needs confirmation with the BAIP commercial and tooling teams

Are AI Units (BAIP runtime billing) and Hyperspace (development tool billing) entirely separate cost streams with no shared tracking or reporting? Is there any proxy equivalent to Hyperspace for monitoring deployed agent token costs from a SAP operator perspective — outside of the customer-visible AI Units billing? See [Open Questions](open-questions-9d98d9f.md) Q17.

### No Engagement Baseline Measurement

No FDE engagement has yet measured build-time token efficiency systematically. Establishing this baseline — even informally in the next two or three engagements — should be a CIS T&O priority.

### Runtime Monitoring in Topology A Requires Customer Consent

Access to per-invocation runtime telemetry in Topology A (customer-managed tenant) requires the customer's explicit consent. The runtime efficiency design is complete — but it is not deployable in Topology A engagements until the consent question in Discovery has been resolved and the legal basis established.


<a name="loioab4fe535cf8743a8997cff355fce1997__section_quick_wins"/>

## Recommended Quick-Wins Table

<table>
<caption>Evidence-Based Quick Wins for FDE Teams</caption>
<tr>
<th valign="top">Technique</th>
<th valign="top">L-Level</th>
<th valign="top">Evidence</th>
<th valign="top">Effort</th>
<th valign="top">Build-Time</th>
<th valign="top">Runtime</th>
</tr>
<tr>
<td valign="top">

Prompt caching (stable content first; calibrate TTL)

</td>
<td valign="top">

L3

</td>
<td valign="top">

Measured: 84% cost reduction (IBD project)

</td>
<td valign="top">

Low — design convention in sapOrchestration

</td>
<td valign="top">

✓

</td>
<td valign="top">

✓

</td>
</tr>
<tr>
<td valign="top">

MCP / tool gating by solution type

</td>
<td valign="top">

L1

</td>
<td valign="top">

Estimated: ~10K tokens/turn saved

</td>
<td valign="top">

Low — APM setup

</td>
<td valign="top">

✓

</td>
<td valign="top">

✓

</td>
</tr>
<tr>
<td valign="top">

Session handoff notes at phase boundaries

</td>
<td valign="top">

L5

</td>
<td valign="top">

Research-backed (context rot; 11/13 models degraded at 32K)

</td>
<td valign="top">

Low — process convention

</td>
<td valign="top">

✓

</td>
<td valign="top">

(partial)

</td>
</tr>
<tr>
<td valign="top">

Code node for deterministic steps (L0 applied)

</td>
<td valign="top">

L0

</td>
<td valign="top">

Design principle — no token cost for deterministic logic

</td>
<td valign="top">

Low-Medium — requires conscious workflow review

</td>
<td valign="top">

—

</td>
<td valign="top">

✓

</td>
</tr>
<tr>
<td valign="top">

Compact JSON output format in sapOrchestration

</td>
<td valign="top">

L2

</td>
<td valign="top">

Engineering principle

</td>
<td valign="top">

Low — configuration choice

</td>
<td valign="top">

✓

</td>
<td valign="top">

✓

</td>
</tr>
<tr>
<td valign="top">

Skill / agent routing by deterministic key

</td>
<td valign="top">

L1 + L5

</td>
<td valign="top">

Estimated: eliminates 10K+ tokens per mis-selection

</td>
<td valign="top">

Low — APM + CLAUDE.md setup

</td>
<td valign="top">

✓

</td>
<td valign="top">

✓

</td>
</tr>
<tr>
<td valign="top">

Per-project APM setup (CIS Starter Kit apm.yml)

</td>
<td valign="top">

L1

</td>
<td valign="top">

Structural — prevents anti-pattern accumulation

</td>
<td valign="top">

Medium — initial setup; low thereafter

</td>
<td valign="top">

✓

</td>
<td valign="top">

—

</td>
</tr>
<tr>
<td valign="top">

Cost per correct outcome metric

</td>
<td valign="top">

L6

</td>
<td valign="top">

Aligns with value-based contract logic

</td>
<td valign="top">

Medium — requires measurement instrumentation

</td>
<td valign="top">

✓

</td>
<td valign="top">

✓

</td>
</tr>
</table>

**Recommendation for a new FDE engagement starting Develop**: implement the first three rows in week 0 of Develop as part of the agentic project setup — they require no special tooling, only conscious configuration choices. Add the fourth and fifth rows during workflow design review. Track the cost per correct outcome metric from the first EAC deployment.

**Related Information**  


[CIS Delivery Efficiency and Reusable Components](reusable-components-62438fe.md)

[Agent Quality Framework](agent-quality-framework-58e930e.md)

[Supervisor Agent Pattern](supervisor-agent-pattern-fa25f56.md)

[Production Operations](production-operations-79a39ed.md)

[Observe and Analyze \(AI Agent Hub\)](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/1fdc1df35d414cd6bf2ed15ec8a164f4.html)
