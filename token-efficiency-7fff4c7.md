<!-- loio7fff4c76d0c240ecbff86e3e9f2f4c90 -->

# Token Efficiency

Token efficiency in Industry AI delivery requires deliberate management of two entirely separate cost pools: build-time tokens consumed by FDE engineers during the sprint, and runtime tokens consumed by deployed agents processing business transactions. These pools have different owners, different measurement mechanisms, and different optimization levers.



<a name="loio7fff4c76d0c240ecbff86e3e9f2f4c90__section_two_pools"/>

## Two Cost Pools

**Token Cost Pool Comparison**


<table>
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
<tr>
<td valign="top">

Build-time

</td>
<td valign="top">

FDE engineers using Claude Code and AI coding tools during the sprint

</td>
<td valign="top">

Hyperspace LLM Proxy \(SAP-internal only\)

</td>
<td valign="top">

CIS / FDE team

</td>
</tr>
<tr>
<td valign="top">

Runtime

</td>
<td valign="top">

Deployed Industry AI agents processing business transactions for the customer

</td>
<td valign="top">

BAIP AI Units + OpenTelemetry \(built into BAIP harness\)

</td>
<td valign="top">

Customer \(billing\) + CIS \(operational monitoring\)

</td>
</tr>
</table>

The Hyperspace LLM Proxy is an SAP-internal tool and is not available for deployed agents running in a customer's BAIP tenant at runtime. Any efficiency measurement of production agents relies exclusively on the BAIP-native observability stack. See [Orchestration — SAP AI Core](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/8d022355037643cebf775cd3bf662cc5.html) for the runtime model service.



<a name="loio7fff4c76d0c240ecbff86e3e9f2f4c90__section_l0_l6"/>

## SAP's L0–L6 Token Optimization Framework

SAP's internal Token Optimization Cheat Sheet defines seven levels of optimization, numbered L0–L6. The framework progresses from the cheapest and most impactful interventions to governance-layer instrumentation:

**L0–L6 Optimization Framework**


<table>
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
<tr>
<td valign="top">

L0

</td>
<td valign="top">

ELIMINATE

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

INPUT CONTROL

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

OUTPUT CONTROL

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

CACHING

</td>
<td valign="top">

Place stable, rarely-changing content at the beginning of prompts to exploit cache reads; stable `CLAUDE.md` for build-time sessions

</td>
</tr>
<tr>
<td valign="top">

L4

</td>
<td valign="top">

MODEL ROUTING

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

AGENT CAPS

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

GOVERNANCE

</td>
<td valign="top">

Track cost per correct outcome, not per token; establish measurement baselines; set accountability structures

</td>
</tr>
</table>



<a name="loio7fff4c76d0c240ecbff86e3e9f2f4c90__section_build_time"/>

## Build-Time Efficiency

### Per-Project Agentic Setup

The single highest-impact build-time change is a per-project, reproducible agentic configuration — a `per-project apm.yml` \(Agent Package Manager\) that declares which MCP servers are registered, which skills apply, and a project `CLAUDE.md` encoding the project overview, stack constraints, forbidden operations, and pointers to ADRs.

The `apm install` command deploys the declared tool set; `apm compile` generates the `CLAUDE.md`. This gives every engineer on the FDE squad an identical, version-locked starting point.

### Top Four Validated Quick Wins

-   **Prompt Caching \(L3\)** — measured 84% cost reduction in SAP's IBD project \($25.99 → $4.22 per session\) by placing stable system prompt content at the top of every `sapOrchestration` call.

-   **Tool Gating \(L1\)** — register only the MCP servers and tools required for the current phase. The FDE anti-pattern is each engineer starting with all globally installed MCP servers active.

-   **Session Handoff Notes \(L5\)** — at the end of each working session, have the AI coding agent write a structured handoff note summarizing decisions made, open questions, and the next phase's starting point. Re-read the note at the start of the next session rather than reloading the full conversation.

-   **Supervisor as Efficiency Gate** — the supervisor agent eliminates unnecessary LLM calls in the worker's retry loop by intercepting failed outputs before they trigger a full re-reasoning cycle.



<a name="loio7fff4c76d0c240ecbff86e3e9f2f4c90__section_runtime"/>

## Runtime Efficiency

For deployed agents, L0 is the most consequential lever: agent workflow design determines which steps require LLM reasoning and which can use deterministic Code nodes. An agent that routes numerical computation to deterministic `sapMcpClient` services requires far fewer `sapOrchestration` calls than one that relies on LLM arithmetic.

**Joule Studio Node Selection for Token Efficiency**


<table>
<tr>
<th valign="top">

Task Type

</th>
<th valign="top">

Correct Node

</th>
<th valign="top">

Do NOT Use

</th>
</tr>
<tr>
<td valign="top">

Numerical calculation with a SAP service equivalent

</td>
<td valign="top">

`sapMcpClient` → authoritative SAP service

</td>
<td valign="top">

`sapOrchestration`: LLM computing the number

</td>
</tr>
<tr>
<td valign="top">

Lookup from a defined authoritative source

</td>
<td valign="top">

`sapMcpClient` → lookup API

</td>
<td valign="top">

LLM retrieval from training data or RAG

</td>
</tr>
<tr>
<td valign="top">

Business rule evaluation with a rules engine

</td>
<td valign="top">

`sapMcpClient` → rules service

</td>
<td valign="top">

LLM rule interpretation

</td>
</tr>
<tr>
<td valign="top">

Contextual judgment — "does this make business sense?"

</td>
<td valign="top">

`sapOrchestration` with a critic prompt

</td>
<td valign="top">

Rigid rule-based check

</td>
</tr>
</table>

> ### Tip:  
> Token efficiency and agent quality are not in tension — they reinforce each other. A well-designed agent that routes deterministic tasks to deterministic tools needs fewer LLM calls, and each LLM call it does make is for a task where reasoning genuinely adds value. This produces both lower token cost and higher accuracy.

**Related Information**  


[SAP AI Core — Orchestration](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/8d022355037643cebf775cd3bf662cc5.html)

[SAP AI Core — Models](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/6440777c196642dbb3171aa5c37b987e.html)

[Agent Quality Framework](agent-quality-framework-af194ba.md)

[CIS CoE Operations and Reusable Components](cis-coe-operations-and-reusable-components-363e20e.md)
