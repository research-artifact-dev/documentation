<!-- loiob51609732a9f4c39b9ab268d1ae3206e -->

# CMJ vs. BYOA Cheat Sheet — Agent Delivery Paths in Joule

This quick reference describes the two orthogonal decision axes for agent delivery in the SAP Joule platform — orchestrator selection and custom-agent runtime location — and provides comparison tables, phased GA status, and decision criteria for FDE teams.

> ### Note:  
> November 2026 framing. "CMJ vs SMJ" is not a single decision — it is **two orthogonal axes**: (1) the orchestrator / Joule activation and (2) where the custom-agent runtime lives. The older bundled "CMJ/BYOA path" label conflated them; keep them separate. In short: the orchestrator is **SMJ by default** (CMJ is deprecated), and the runtime is **BYOA (pro-code) or Joule Studio 2.0 (low-code)** — BYOA is a standard runtime, not a fallback.


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_two_axes"/>

## The Two Axes

**Axis 1 — Orchestrator / Joule Activation.** **SMJ (SAP-Managed Joule, "THE BOX") is the default for all new engagements** — GA since 2026-06-01, the recommended activation method for all customers. **CMJ (Customer-Managed Joule) is deprecated** — a migration target, used only where SMJ is not yet available in-region (sovereign: CN40, NS2, KSA) or for engagements already in progress on CMJ. CMJ now receives only AutoSQL / Knowledge Graph / DPQuery; Joule Work, Joule Studio 2.0, and new LoB onboarding are **out of scope for CMJ**.

**Axis 2 — Custom-Agent Runtime Location.** **BYOA** (custom agent on the customer's BTP subaccount — LangGraph / AG2 / CAP / CrewAI — registering to the SAP-managed Agent Gateway via **A2A**) **or Joule Studio 2.0** (low-code: canvas / CLI). **BYOA is the standard runtime for productization-intent pro-code agents — not a fallback.** Both run within the SMJ context.

**Runtime comparison**

<table>
<thead>
<tr>
<th valign="top">

 

</th>
<th valign="top">

Low-Code Runtime

</th>
<th valign="top">

Pro-Code Runtime

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Build surface

</td>
<td valign="top">

Joule Studio 2.0 (n8n canvas / CLI)

</td>
<td valign="top">

BYOA (LangGraph/AG2/CAP/CrewAI on customer BTP subaccount)

</td>
</tr>
<tr>
<td valign="top">

Registers to

</td>
<td valign="top">

SMJ managed runtime (automatic)

</td>
<td valign="top">

SMJ Agent Gateway via A2A

</td>
</tr>
<tr>
<td valign="top">

When to choose

</td>
<td valign="top">

Simple skills, citizen-dev, rapid POC

</td>
<td valign="top">

Complex / stateful agents with productization intent (the standard FDE case)

</td>
</tr>
</tbody>
</table>

**Two FDE delivery types:** *IAI CAE* (customer-specific first, productize later) → BYOA agent on the customer subaccount + SMJ orchestrator. *Standard Industries Development* (build into SAP standard) → Joule Studio 2.0 on SMJ.

**Default rule:** orchestrator = SMJ for all new engagements; pick the runtime (BYOA vs JS2.0) by complexity/productization intent. Use a **customer-managed CMJ orchestrator** only for a concrete reason — sovereign region where SMJ is unavailable, non-standard Kyma the customer's infra team requires, or an engagement already in progress on CMJ. Document the reason in Discovery and the productization package.


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_phased_ga"/>

## Phased GA Status (November 2026)

GA is **not binary** — it is layered. Do not represent the JS2.0 custom-agent managed runtime as broadly GA without this qualification.

**GA layer status**

<table>
<thead>
<tr>
<th valign="top">

Layer

</th>
<th valign="top">

Status

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

SMJ / THE BOX orchestrator

</td>
<td valign="top">

**GA since 2026-06-01**

</td>
</tr>
<tr>
<td valign="top">

Joule Studio 2.0 build environment

</td>
<td valign="top">

**Phased GA** (fair-use limits apply)

</td>
</tr>
<tr>
<td valign="top">

Custom-agent managed runtime on SMJ

</td>
<td valign="top">

**EAC-gated / select customers only**, free through Dec 2026 — not broadly GA

</td>
</tr>
<tr>
<td valign="top">

Outbound A2A (agent registered + surfaced in Joule)

</td>
<td valign="top">

Available, limited / unidirectional

</td>
</tr>
<tr>
<td valign="top">

Bidirectional A2A for the managed-runtime BYOA path (Joule ↔ custom agent)

</td>
<td valign="top">

**Transitional — not GA** (SAP Architecture Center "transitional state")

</td>
</tr>
<tr>
<td valign="top">

Inbound / external→SAP A2A (third party invokes an SAP agent via ARD / Agent Gateway Consumer)

</td>
<td valign="top">

**Transitional — not GA** (Architecture Center); intended future mechanism; **H1 2027 target**

</td>
</tr>
</tbody>
</table>

**Practical rule for engagements starting now:** architect the **BYOA + outbound-A2A** path (functional today); treat full Joule Studio managed-runtime deployment + bidirectional A2A as an **H1 2027 target**.


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_js20_runtime"/>

## The Joule Studio 2.0 (Low-Code) Runtime on SMJ

Workflow when the custom-agent runtime is Joule Studio 2.0 on the SMJ orchestrator:

1.  **Build** the agent in Joule Studio 2.0 — canvas (n8n), Joule Studio CLI (`jl`), or the A2A Agent Toolkit for pro-code components
2.  **Deploy** via Joule Studio — the managed runtime handles BTP provisioning, auth, and the observability harness automatically
3.  **Integrate with Joule Work** — automatic; no manual capability bundle or `joule deploy` needed
4.  **Govern** via SAP AI Agent Hub — agent appears in the Agent & MCP Registry automatically; IT admin validates and activates it

**What the managed runtime provides automatically (no manual wiring required)**

<table>
<thead>
<tr>
<th valign="top">

Concern

</th>
<th valign="top">

BYOA Pro-Code (Manual)

</th>
<th valign="top">

Joule Studio 2.0 Managed Runtime (Automatic)

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

OAuth2 / IAS authentication

</td>
<td valign="top">

FDE team wires manually

</td>
<td valign="top">

Automatic

</td>
</tr>
<tr>
<td valign="top">

OpenTelemetry harness (TR12)

</td>
<td valign="top">

Manual — OTel to SAP Cloud ALM (or a self-managed collector in sovereign regions)

</td>
<td valign="top">

Automatic

</td>
</tr>
<tr>
<td valign="top">

Agent identity + BAIP registration

</td>
<td valign="top">

Manual capability bundle

</td>
<td valign="top">

Automatic

</td>
</tr>
<tr>
<td valign="top">

Joule Work surface integration

</td>
<td valign="top">

Manual `joule deploy`

</td>
<td valign="top">

Automatic

</td>
</tr>
<tr>
<td valign="top">

AI Agent Hub registry entry

</td>
<td valign="top">

Not automatic

</td>
<td valign="top">

Automatic (IT admin activates)

</td>
</tr>
</tbody>
</table>

> ### Note:  
> **"Not validated" state**: newly deployed JS2.0 agents appear in AI Agent Hub in a "not validated" state until the customer IT admin explicitly activates them. Include this activation step in every productization handover checklist — the agent is deployed and functional, but not in full governance scope until activated.

JS2.0 agents run in a customer-specific SMJ tenant, not a shared SAP environment. Data residency and sovereign-boundary guarantees are maintained per customer; the "SAP-managed" label refers to the operational infrastructure, not data ownership. Agent telemetry is pushed to **SAP Cloud ALM**, an SAP-operated service region-bound to the customer's contracted data-residency region — not an in-tenant store.


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_byoa_runtime"/>

## The BYOA (Pro-Code) Runtime

### How a BYOA Agent Appears in Joule

From the end user's perspective, a BYOA agent looks identical to a native SAP agent — it appears as a Joule Assistant in the Joule sidebar. The wiring model is invisible.

Under the hood: Joule calls an HTTPS endpoint on the customer's BTP subaccount (CF or Kyma) via the **A2A protocol**. Under the SMJ default, the BYOA agent **registers to the SAP-managed Agent Gateway** via A2A; in a sovereign/in-progress **CMJ-orchestrator** case, the FDE team instead wires it manually via `joule deploy`.

**A2A direction — mind the phased-GA caveat:**

-   **Outbound (Joule → custom agent)**: available today; the standard BYOA path.
-   **Bidirectional for the managed-runtime BYOA path**: **transitional — not GA**. Architect for outbound + async callbacks now.
-   **Inbound / external→SAP (third party invokes an SAP agent)**: **transitional — not GA** (SAP Architecture Center). The intended future mechanism is ARD-based discovery + SAP-managed formations (Identity → Trust → Authorization) + Agent Gateway Consumer registration; **H1 2027 target**.

### What the FDE Team Must Wire Manually

Items the managed runtime handles automatically for Joule Studio 2.0, which the FDE team handles manually on the BYOA pro-code runtime:

**Manual wiring requirements for BYOA**

<table>
<thead>
<tr>
<th valign="top">

What

</th>
<th valign="top">

Why It Matters

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

**Auth** (OAuth2 CC or IAS)

</td>
<td valign="top">

Joule must authenticate to call the agent

</td>
</tr>
<tr>
<td valign="top">

**Public Agent Card endpoint**

</td>
<td valign="top">

Joule reads `/.well-known/agent.json` during registration — must be publicly reachable

</td>
</tr>
<tr>
<td valign="top">

**A2A registration / capability bundle**

</td>
<td valign="top">

Registers the agent to SMJ (via A2A) or, for a CMJ orchestrator, the 4-file `joule deploy` bundle

</td>
</tr>
<tr>
<td valign="top">

**Observability**

</td>
<td valign="top">

Not automatic — wire OTel to SAP Cloud ALM (or a self-managed OTLP collector in sovereign regions such as NS2 where Cloud ALM is unavailable)

</td>
</tr>
<tr>
<td valign="top">

**Regional orchestrator check**

</td>
<td valign="top">

If SMJ is not available in-region, a CMJ orchestrator (separate prerequisite) is required

</td>
</tr>
</tbody>
</table>

### The Three Steps for BYOA Delivery

**BYOA delivery steps**

<table>
<thead>
<tr>
<th valign="top">

Step

</th>
<th valign="top">

What Happens

</th>
<th valign="top">

Gate to Pass Before Moving On

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

**1 — Local agent**

</td>
<td valign="top">

Scaffold agent; answer real A2A messages with mock/test data

</td>
<td valign="top">

`SendMessage` returns a valid response locally

</td>
</tr>
<tr>
<td valign="top">

**2 — Deploy to BTP**

</td>
<td valign="top">

Push to Cloud Foundry or Kyma; public HTTPS endpoint live

</td>
<td valign="top">

Agent Card reachable at public URL; smoke test passes over HTTPS

</td>
</tr>
<tr>
<td valign="top">

**3 — Register to Joule**

</td>
<td valign="top">

Register to SMJ Agent Gateway via A2A (or `joule deploy` for a CMJ orchestrator); test from Joule sidebar

</td>
<td valign="top">

Capability shows ACTIVE; Joule routes a real prompt to the agent

</td>
</tr>
</tbody>
</table>

> ### Caution:  
> **60-second constraint (both runtimes):** Joule expects a synchronous response within 60 seconds. Agent workflows involving chained LLM calls or external data fetches must fit within this window or use the async A2A pattern (push notifications). Measure p99 latency under realistic load before assuming synchronous is viable.


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_ownership_summary"/>

## Ownership Summary

**Ownership by runtime**

<table>
<thead>
<tr>
<th valign="top">

 

</th>
<th valign="top">

Joule Studio 2.0 Runtime (on SMJ)

</th>
<th valign="top">

BYOA Pro-Code Runtime

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Writes the agent

</td>
<td valign="top">

FDE team

</td>
<td valign="top">

FDE team

</td>
</tr>
<tr>
<td valign="top">

Manages the Joule/BAIP account

</td>
<td valign="top">

SAP (customer subscribes to SMJ)

</td>
<td valign="top">

SAP (SMJ); customer IT only if a CMJ orchestrator is used

</td>
</tr>
<tr>
<td valign="top">

Wires auth, observability, Joule registration

</td>
<td valign="top">

**Automatic (managed runtime)**

</td>
<td valign="top">

FDE team (manual)

</td>
</tr>
<tr>
<td valign="top">

Operates post-handover

</td>
<td valign="top">

Customer IT + CIS T&O (via supervisor mechanism)

</td>
<td valign="top">

Customer IT + CIS T&O (via supervisor mechanism)

</td>
</tr>
<tr>
<td valign="top">

IP & productization

</td>
<td valign="top">

SAP — standard FDE model, no change

</td>
<td valign="top">

SAP — standard FDE model, no change

</td>
</tr>
</tbody>
</table>


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_deployment_landscape"/>

## Current Deployment Landscape

```text
Orchestrator axis:  SMJ (default, GA 2026-06-01) ───────────────► all new engagements
                    CMJ (deprecated) ──► only sovereign / in-progress, where SMJ is unavailable

Runtime axis (within SMJ):
  Joule Studio 2.0 (low-code):  build in JS2.0 → managed runtime (automatic auth/OTel/registry)
                                 → automatic Joule Work integration    [managed runtime EAC-gated]
  BYOA (pro-code):              build on BTP (CF/Kyma) → register to SMJ Agent Gateway via A2A
                                 → appears as a Joule Assistant         [standard for productization]

Productized (any path):         FDE productizes → agent becomes SAP standard
                                 → appears in Joule Work / SAP for Me for all customers
```


<a name="loiob51609732a9f4c39b9ab268d1ae3206e__section_status_open_items"/>

## Status and Open Items

**Feature status summary**

<table>
<thead>
<tr>
<th valign="top">

Item

</th>
<th valign="top">

Status

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

SMJ orchestrator

</td>
<td valign="top">

**GA since 2026-06-01** — default for all new engagements

</td>
</tr>
<tr>
<td valign="top">

Joule Studio 2.0 build environment

</td>
<td valign="top">

**Phased GA** (fair-use limits)

</td>
</tr>
<tr>
<td valign="top">

JS2.0 custom-agent managed runtime

</td>
<td valign="top">

**EAC-gated / select customers**, free through Dec 2026 — not broadly GA; full managed-runtime deployment is an H1 2027 target

</td>
</tr>
<tr>
<td valign="top">

CMJ (customer-managed orchestrator)

</td>
<td valign="top">

**Deprecated** — migration target; use only where SMJ is unavailable in-region or for in-progress engagements; document the reason

</td>
</tr>
<tr>
<td valign="top">

BYOA (pro-code runtime)

</td>
<td valign="top">

**Standard** runtime for productization-intent agents — registers to SMJ via A2A; **not a fallback**

</td>
</tr>
<tr>
<td valign="top">

Bidirectional A2A (managed-runtime BYOA path)

</td>
<td valign="top">

**Transitional — not GA** (Architecture Center); architect for outbound + async callbacks

</td>
</tr>
<tr>
<td valign="top">

Inbound / external→SAP A2A (ARD / Agent Gateway Consumer)

</td>
<td valign="top">

**Transitional — not GA** (Architecture Center); intended future mechanism; **H1 2027 target**

</td>
</tr>
<tr>
<td valign="top">

Formal IAI/CIS guidance on delivery path

</td>
<td valign="top">

Effective guidance = SMJ orchestrator default; BYOA/JS2.0 runtime by complexity; CMJ only when specifically needed. Monitor the IAI/CIS alignment channel for formal policy confirmation

</td>
</tr>
</tbody>
</table>

**Related Information**  


[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "SAP's AI-first development environment for building and deploying AI agents, intelligent apps, and automations.") :arrow_upper_right:

[Developing with Joule Studio CLI](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/a7bbbde0cf3b48f4a82a4403577a6b2c.html "Build Joule Studio solutions from a local IDE using the jl CLI, with full control over code structure and CI/CD integration.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub — discover, govern, and monitor AI agents across your enterprise landscape.") :arrow_upper_right:
