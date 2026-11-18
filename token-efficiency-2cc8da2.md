<!-- loio2cc8da21423848b6806cd02e8544d453 -->

# Token Efficiency in Industry AI Delivery

Token efficiency in Industry AI delivery spans two separate cost pools — build-time (FDE engineers using AI coding tools during the sprint) and runtime (deployed agents processing business transactions) — each with different measurement mechanisms, owners, and optimization levers. Deliberate token use is measurable, and the correct efficiency metric is cost per correct outcome, not raw token throughput.


<a name="loio2cc8da21423848b6806cd02e8544d453__section_why_matters"/>

## Why Token Efficiency Matters

### The Shift from Subsidy to Accountability

The economics of AI development have changed. SAP's Token Consumption Governance Framework, which went live on 17 August 2026 via the Hyperspace LLM Proxy, made this shift concrete for SAP-internal tooling. Token budgets and consumption plans now apply to developer tools (Claude Code, Codex, Pi, opencode) at the individual and team level.

This is not a signal to use less AI — it is a signal that token consumption must be deliberate and measurable. For CIS and FDE teams, deliberate means: knowing where tokens are spent, having techniques to reduce waste, and framing efficiency not as token count but as value delivered per token spent.

### Two Cost Pools with Different Owners

**Table: Build-Time vs. Runtime Cost Pools**

<table>
<thead>
<tr>
<th valign="top">

Cost Pool

</th>
<th valign="top">

What It Covers

</th>
<th valign="top">

How It Is Measured

</th>
<th valign="top">

Who Owns It

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

> ### Note:  
> The runtime pool decomposes at deployment into **three separate, non-consolidatable billing pools — AI Units + BTPEA credits + BDC capacity units** — that map to different contract line items. See the Runtime Efficiency section for the breakdown and its value-case implications.

**Critical constraint**: the Hyperspace LLM Proxy is an SAP-internal tool. It is not available for deployed agents running in a customer's BAIP tenant at runtime. Any efficiency measurement of production agents relies exclusively on the BAIP-native observability stack: AI Units for commercial billing and OpenTelemetry traces for operational monitoring (see [Production Operations](production-operations-168fd42.md)).

CIS carries accountability for both pools: build-time efficiency protects the sprint budget; runtime efficiency protects the customer's value case.


<a name="loio2cc8da21423848b6806cd02e8544d453__section_framework"/>

## SAP's L0–L6 Token Optimization Framework

SAP's internal Token Optimization Cheat Sheet defines seven levels of optimization, numbered L0–L6. The framework progresses from the cheapest and most impactful interventions (L0) to governance-layer instrumentation (L6).

**Table: L0–L6 Token Optimization Levels**

<table>
<thead>
<tr>
<th valign="top">

Level

</th>
<th valign="top">

Name

</th>
<th valign="top">

Core Action

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

The framework applies to both cost pools but with different emphasis:

-   For **build-time** (FDE engineers), L0–L5 are practical daily habits and tool configuration choices. L6 is the engagement-level measurement question.
-   For **runtime** (deployed agents), L0 is the most consequential — agent workflow design determines which steps require LLM reasoning and which can use deterministic Code nodes. L1–L3 are encoded into the Joule Studio workflow design and the `sapOrchestration` configuration.


<a name="loio2cc8da21423848b6806cd02e8544d453__section_buildtime"/>

## Build-Time Efficiency: FDE Engineer Guidelines

### Per-Project Agentic Setup (L1 + L5)

The single highest-impact build-time change is establishing a per-project, reproducible agentic configuration rather than relying on individual engineers' global tool installations.

**The anti-pattern**: each engineer installs whatever MCP servers and skills they prefer globally. Every session starts with a different tool set, different context, and no shared project conventions. Token waste is structurally guaranteed.

**The better pattern**: a per-project `apm.yml` (Agent Package Manager, a Microsoft OSS tool for managing agentic project configurations) that declares:

-   Which MCP servers are registered for this project type (not all MCP servers globally)
-   Which skills apply to this engagement's workflow
-   A project `CLAUDE.md` encoding the project overview, stack constraints, forbidden operations, and pointers to ADRs and specs

The `apm install` command deploys the declared tool set; `apm compile` generates the `CLAUDE.md`. This gives every engineer on the FDE squad an identical, version-locked starting point.

The CIS Starter Kit ([Reusable Components and Reference Implementations](reusable-components-4ff1667.md)) is the natural home for an `apm.yml` template per common engagement type.

> ### Note:  
> `apm` is a Microsoft-originated, MIT-licensed open-source tool (github.com/microsoft/apm) adopted by SAP — not an SAP-built tool. No confirmed SAP Software Hub / PPMS registration could be found. Build-time use on SAP-internal CI and developer workstations is fine. **Customer-environment deployment is uncleared until a PPMS entry is confirmed** — the trigger is whether the `apm` binary or its generated output is bundled into a customer deliverable. For a binding use-scope ruling, contact **SAP_CIS_ATQ@sap.com** before the engagement starts.

**What belongs in the project `CLAUDE.md`:**

**Table: CLAUDE.md Sections and Token Impact**

<table>
<thead>
<tr>
<th valign="top">

Section

</th>
<th valign="top">

Content

</th>
<th valign="top">

Token Impact

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

Context management is one of the most important practices in agentic engineering. Loading only what the current phase needs is the practical implementation of L1 and L5.

### Top Four Quick Wins from Validated Evidence

The following four techniques have the strongest evidence base from SAP-internal measurements and apply directly to FDE sprint workflows.

#### Prompt Caching (L3) — Measured 84% Cost Reduction

**Evidence**: IBD (Intelligent Business Documentation) project measured $25.99 → $4.22 per session (84% cost reduction) by placing stable system prompt content at the top of every `sapOrchestration` call and setting the Anthropic cache read rate (0.1× base input price) for those tokens.

**FDE application**: the agent's system prompt, domain context, and tool descriptions change rarely within a sprint phase. Place static content first; dynamic per-invocation context last. Choose cache TTL based on the actual inter-turn gap for this workflow — the default 1-hour TTL is wrong for many FDE workflows, where sessions may run for days.

**Where it applies**: any `sapOrchestration` node with a substantial system prompt. Especially high-value for agents that share a common domain knowledge base across many invocations.

#### MCP & Tool Gating (L1) — ~10K Tokens Saved Per Turn

**Evidence**: registering only solution-type-relevant MCP tools (instead of all available tools) saves approximately 10,000 tokens per agent turn, and 500K+ tokens over a 50+ turn session. Deterministic routing by solution type selects the correct tool subset without dynamic selection overhead.

**FDE application**: every Joule Studio session should register only the SAP systems and MCP tools relevant to this engagement's use case. For an asset management agent, do not register procurement or HR MCP servers. Establish a routing key (solution type) that determines which tool subset loads.

> ### Caution:  
> Do not apply this to supervisor workflows that need cross-domain visibility. The efficiency gain from narrow tool sets applies to specialist worker agents, not to generic monitoring or support relay workflows.

#### Session Handoff at Phase Boundaries (L5) — Compounds with Caching

**Why it works**: starting a fresh session at each sprint phase boundary (Discovery → Develop → Productization, or within Develop at major artifact milestones) eliminates re-sent upstream transcript from the previous phase. It also enables per-phase model routing.

**The context rot risk**: model performance degrades as input length increases, often before the nominal context limit is reached. Research shows 11 of 13 models showed degraded performance at 32K tokens despite a 128K claimed context window. The practical quality ceiling is approximately 50% of the nominal context window.

**FDE application**: plan explicit session boundaries as part of the sprint workflow design. At each boundary, write a structured handoff note covering: decisions taken, artifacts produced, open items, and the entry point for the next session.

**Spec-driven development** (the multi-phase pipeline: requirements → architecture → module specs → implementation plan → code generation, each in its own session) is the architectural expression of this principle at the engagement level.

#### Skill Loading and Deterministic Routing (L1 + L5)

**Why it matters**: without deterministic skill routing, the agent selects skills via LLM reasoning. In production, this leads to mis-selection — the wrong skill is invoked, which typically costs 10,000+ tokens before the error is detected and corrected.

**FDE application**: define a routing key (e.g., solution type: asset management, procurement, manufacturing) at the start of each session. Use this key to load only the skills relevant to this engagement type. Core skills (project conventions, safety rules) are always loaded; domain-specific skills are routed by key. This is the same routing key used in MCP gating — one key, two uses.

**Pairs with APM**: the `apm.yml` per-project setup can encode which skills are loaded for each solution type, making routing deterministic by default.

### Input Compression: LLMLingua and Caveman

Two validated input compression techniques are available for reducing input token costs on non-cached, non-structured portions of prompts. Both have been benchmarked in a SAP-internal study (ADR 011, 30 enterprise cases).

**Table: Input Compression Technique Comparison**

<table>
<thead>
<tr>
<th valign="top">

Technique

</th>
<th valign="top">

Input Reduction

</th>
<th valign="top">

Quality Score

</th>
<th valign="top">

Numeric Recall

</th>
<th valign="top">

Latency

</th>
<th valign="top">

Caching-Compatible?

</th>
</tr>
</thead>
<tbody>
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

< 1 second

</td>
<td valign="top">

Yes

</td>
</tr>
</tbody>
</table>

**Operating guidance:**

-   Use **Caveman** as the default: lossless, fast, and caching-compatible. 29% input reduction with zero quality degradation.
-   Use **LLMLingua at 50% retention** only for non-numerical, non-financial prompts where a 9% numeric recall drop is acceptable — **never** for batch release quantities, pricing, or financial postings.
-   **Never use LLMLingua at 25% retention** — quality falls below threshold (0.882).
-   **Critical constraint — caching incompatibility**: input compression defeats prompt caching if applied to cached prompt segments. Only compress the non-cached portion of each prompt.

Both ADR 011 (compression) and ADR 014 (startup context) are in Draft/In-Review status; treat as recommended practices rather than mandated standards until formal adoption.

### Agent Startup Context — AGENTS.md Pattern

Providing a structured startup context file (`AGENTS.md` or `CLAUDE.md`) at the project root — rather than relying on the agent to explore the codebase at session start — reduces token consumption per session. An external Codex benchmark (ADR 014, 124 paired tasks) measured the following:

**Table: AGENTS.md Efficiency Impact**

<table>
<thead>
<tr>
<th valign="top">

Metric

</th>
<th valign="top">

Change with AGENTS.md

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

The trade-off is deliberate: spend tokens once on startup context to avoid repeated exploratory tool calls throughout the session. The runtime improvement (28.64% lower) is especially significant for FDE sprint work where engineer time — not just token cost — is the constrained resource.

The `AGENTS.md` name is the convention for multi-agent setups (e.g., OpenAI Codex and other agentic frameworks); `CLAUDE.md` is the Claude Code convention. Both patterns apply the same principle and produce the same efficiency gains.

### Measurement During the Sprint: Project-Level Cost Tracking

Token efficiency during the sprint should be measured at the project level, not the individual level. The HAI proxy (Hyperspace LLM Proxy) tracks consumption per individual user via SSO identity — there is no built-in project grouping today.

**Background — what the HAI proxy does and does not expose**: the proxy captures per-request data: user identity (SSO/OIDC), model, token counts (input / output / cache read / cache write), timestamp, and EUR cost. There is no programmatic API to query spending data, no project identifier attached to requests, and no team or workspace grouping in the current spending layer. Shared proxies and shared API keys are explicitly prohibited.

**Option A — Project-level API key (shared key per engagement)**: not feasible without significant HAI proxy backend work. Every request is attributed to the authenticated user via SSO, not the API key.

**Option B — Project-level tracking inside the HAI proxy backend**: the correct CIS action is to **feed project-level attribution as a formal requirement into the HAI proxy team's workspace roadmap**, since FDE is a primary driver of the business case. Not feasible near-term, but the right structural solution.

**Option C — Local accounting sidecar (recommended near-term approach)**:

A lightweight tool that sits between AI coding tools and the HAI proxy, intercepts responses, extracts token counts from the standard API response body, and logs them with project context:

```
AI Tool (Claude Code)
  → localhost:6656  [local project accounting sidecar]
    → localhost:6655  [HAI proxy — unchanged]
      → Hyperspace backend
```

The Anthropic API response always includes `usage.input_tokens`, `usage.output_tokens`, `usage.cache_read_input_tokens`, and `usage.cache_creation_input_tokens` in the JSON body. The sidecar forwards all traffic unchanged, reads token counts from each response, and appends a log line: `timestamp | project_id | model | tokens | EUR_estimate`.

**Project ID auto-detection**: the sidecar reads the project identifier from the engagement's git repository root (e.g., a `.hai-project` file or the existing `CLAUDE.md` project name). When Claude Code is invoked from within the engagement repo, the project context is resolved automatically.

> ### Note:  
> Auto-detection assumes a **standard CIS FDE project scaffold** — a conventional repo root carrying a `.hai-project` marker or a `CLAUDE.md` with the project name. That scaffold does not yet exist as a CIS standard. Until it does, the **manual fallback** is for each engineer to set the project ID explicitly in the sidecar config once per engagement.

**Recommendation**: implement Option C as a CIS-internal tooling initiative; feed Option B into the HAI proxy workspace roadmap as a formal FDE requirement.

Until a sidecar or workspace feature is available, the manual fallback is: each engineer records their HAI proxy monthly total at sprint start and sprint end, and reports the delta to a shared engagement log.


<a name="loio2cc8da21423848b6806cd02e8544d453__section_runtime"/>

## Runtime Efficiency: Deployed Agents

### Different Measurement, Different Levers

Once an Industry AI agent is in production in a customer's BAIP tenant, the token efficiency measurement shifts entirely to the BAIP-native observability stack:

-   **AI Units** (commercial): the per-Action billing model visible in the customer's SAP subscription. The contractual metering unit is **0.02 AI Units per Agent Action** (1 Agent Action = 1 LLM call + 1 simple tool invocation). See [Production Operations](production-operations-168fd42.md) for the three-tier step model (Basic / Standard / Advanced) for detailed cost modelling.
-   **OpenTelemetry traces** (operational): per-invocation token counts from the generative AI hub, already collected by the BAIP harness automatically.

**Three runtime billing dimensions**: runtime costs for a deployed FDE agent span three separate billing pools:

**Table: Runtime Billing Pools**

<table>
<thead>
<tr>
<th valign="top">

Billing Pool

</th>
<th valign="top">

What It Covers

</th>
<th valign="top">

Billing Unit

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

These three dimensions cannot be consolidated into a single AI Units figure. When building the FDE value case, confirm which pools the customer has contracted capacity for.

For **Topology A deployments** (customer-managed tenant), access to runtime token data requires customer consent — the same consent required for operational monitoring generally.

### L0 at Agent Design Time

The most impactful runtime efficiency decision is made at design time: does this step need LLM reasoning, or can a deterministic Code node do it?

In Joule Studio (n8n), every step that passes through `sapOrchestration` or the AI Agent node costs tokens. Every step that runs in a Code node costs compute — but not LLM tokens. The design rule:

> **Use a Code node for any step whose logic can be expressed as deterministic rules. Use `sapOrchestration` only when the step genuinely requires language understanding, reasoning across ambiguous inputs, or natural language generation.**

**Table: Common Over-Use of LLM Nodes**

<table>
<thead>
<tr>
<th valign="top">

Step

</th>
<th valign="top">

Often Done with LLM

</th>
<th valign="top">

Better Approach

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

### Joule Studio Design Choices for Runtime Efficiency

**Output format discipline (L2)**: configure `sapOrchestration`'s `outputSchema` to return compact, structured JSON rather than verbose natural language. Set `max_tokens` in the output schema to prevent verbosity drift — an output ceiling that reflects the actual maximum expected output size.

**MCP call batching (L1)**: where the workflow makes multiple reads from the same SAP system in one turn, batch them via the `sapMcpClient` batch call pattern rather than sequential single-call reads.

**Context scoping per phase (L1 + L5)**: agent workflows that accumulate conversation history across many turns within a session should apply proactive compaction before the context degrades. Apply the session compaction trigger at 50% of nominal context window.

### The Supervisor as Efficiency Gate

The supervisor pattern (see [Supervisor Agent Pattern](supervisor-agent-pattern-0d90e5f.md)) has a token efficiency benefit: the supervisor intercepts output before write-back. If an agent has produced a mis-formed or off-range output after excessive token expenditure, the supervisor rejects it before it triggers a human approval flow — containing the cost of the failure to a single invocation rather than letting the error propagate into the human loop.

Supervisor Workflow B (Telemetry Monitor) already monitors token anomalies as part of its scheduled monitoring cycle. The specific signals — per-invocation tokens vs. rolling baseline, runaway detection at 5× p99, daily budget threshold — are defined in [Production Operations](production-operations-168fd42.md).


<a name="loio2cc8da21423848b6806cd02e8544d453__section_quality_efficiency"/>

## The Quality–Efficiency Connection

### Token Cost as a Measurable Sub-Dimension of Tool Call Efficiency

Tool call efficiency — "did the agent reach the goal with the minimum necessary tool calls?" — becomes measurable in monetary terms through token cost. An agent that reaches the same business outcome with half the tool calls, half the context re-sent, and half the output verbosity is better — not just cheaper. Token efficiency is not in tension with quality; at the operational level, high token consumption is frequently a quality symptom:

**Table: Token Anomaly Signals and Quality Root Causes**

<table>
<thead>
<tr>
<th valign="top">

Token Anomaly Signal

</th>
<th valign="top">

Likely Quality Root Cause

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

### Goodhart's Law Warning

Minimizing token count while ignoring task completion quality is the wrong optimization target. An agent that reaches correct outcomes with fewer tokens is genuinely better. An agent that reaches fewer correct outcomes but uses fewer tokens is worse, regardless of what the cost dashboard shows.

The correct efficiency metric — from SAP's L6 governance level — is **cost per correct outcome**: the total LLM cost divided by the number of outputs that were correct, approved, and committed to the system of record.

For FDE engagements, cost per correct outcome can be approximated as:

```
Cost per correct outcome = Total AI Units consumed
                           ──────────────────────────────────────────
                           Outputs committed to system of record (Dimension 2 process KPI)
```

This metric should be included in the FDE value case alongside the business outcome metrics.

At **build-time** the same L6 logic applies with a different numerator and denominator — **Hyperspace EUR spent / number of correct (accepted, merged) build artifacts**:

```
Cost per correct outcome (build-time) = Total Hyperspace EUR spent
                                        ────────────────────────────────────────
                                        Correct build artifacts (accepted / merged)
```


<a name="loio2cc8da21423848b6806cd02e8544d453__section_gaps"/>

## Known Gaps and Open Questions

The following gaps limit the current ability to implement full token efficiency measurement and governance for CIS engagements.

**Q16 — Hyperspace LLM Proxy per-project cost tracking**: Hyperspace LLM Proxy costs can currently only be tracked and reported at the individual developer level, not at the project or FDE engagement level. Without project-level grouping, the build-time cost pool cannot be managed as an engagement expense. Per-engagement baseline measurement requires this capability. *Status: Open — needs verification with the Hyperspace/DTS tooling team.*

**Q17 — AI Units vs. Hyperspace: relationship and runtime proxy availability**: whether AI Units (BAIP runtime billing) and Hyperspace (development tool billing) are entirely separate cost streams with no shared tracking or reporting, and whether any proxy equivalent to Hyperspace is available for monitoring deployed agent token costs from a SAP operator perspective. *Status: Open — needs confirmation with the BAIP commercial and tooling teams.*

**No engagement baseline measurement**: no FDE engagement has yet measured build-time token efficiency systematically. Establishing a baseline measurement approach — even informally in the next two or three engagements — should be a CIS T&O priority.

**Runtime monitoring in Topology A requires customer consent**: as noted above and in [Production Operations](production-operations-168fd42.md), access to per-invocation runtime telemetry in Topology A requires the customer's explicit consent. The runtime efficiency design is complete — but it is not deployable in Topology A engagements until the consent question in Discovery has been resolved and the legal basis established.


<a name="loio2cc8da21423848b6806cd02e8544d453__section_quickwins"/>

## Recommended Quick-Wins Table

The following table summarizes the techniques with the strongest evidence-to-effort ratio, organized for FDE teams starting a new engagement.

**Table: Token Efficiency Quick-Wins**

<table>
<thead>
<tr>
<th valign="top">

Technique

</th>
<th valign="top">

L-Level

</th>
<th valign="top">

Evidence

</th>
<th valign="top">

Effort

</th>
<th valign="top">

Build-Time

</th>
<th valign="top">

Runtime

</th>
</tr>
</thead>
<tbody>
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
</tbody>
</table>

**Recommendation for a new FDE engagement starting Develop**: implement the first three rows in week 0 of Develop as part of the agentic project setup — they require no special tooling, only conscious configuration choices. Add the fourth and fifth rows during workflow design review. Track the cost per correct outcome metric from the first EAC deployment.

**Related Information**  


[Generative AI Hub with SAP AI Core](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/7db524ee75e74bf8b50c167951fe34a5.html "The generative AI hub enables access to and orchestration of generative AI models within a governed, enterprise-ready runtime on SAP AI Core.") :arrow_upper_right:

[SAP AI Core — Compliance and Security](https://help.sap.com/docs/AI_CORE/f7c0f89219354ee892aaec34a488d2e3/d0be812c81734355ade9dc94638870cf.html "SAP AI Core security measures including data protection, compliance standards, and cloud security certifications.") :arrow_upper_right:

[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "SAP's AI-first development environment for building and deploying AI agents, intelligent apps, and automations.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub — discover, govern, and monitor AI agents across your enterprise landscape.") :arrow_upper_right:
