<!-- loioa0231c5e241a4bb88a953b3fae10b9cc -->

# CMJ vs. BYOA: FDE Delivery Decision Guide

Use this guide to select the correct agent delivery path for Joule engagements. It clarifies the two orthogonal delivery axes — orchestrator choice and custom-agent runtime location — and replaces the legacy "CMJ/BYOA path" label that conflated them.


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_two_axes"/>

## The Two Axes

Agent delivery in Joule is governed by two independent decisions that must be made separately.

**Axis 1 — Orchestrator / Joule Activation**

**SMJ (SAP-Managed Joule, "THE BOX") is the default orchestrator for all new engagements** — generally available since 2026-06-01 and the recommended activation method for all customers. **CMJ (Customer-Managed Joule) is deprecated** and is a migration target only. Use a CMJ orchestrator exclusively in the following cases:

-   SMJ is not yet available in-region (sovereign regions: CN40, NS2, KSA).
-   The engagement is already in progress on CMJ.

CMJ now receives only AutoSQL, Knowledge Graph, and DPQuery updates. Joule Work, Joule Studio 2.0, and new line-of-business onboarding are out of scope for CMJ.

**Axis 2 — Custom-Agent Runtime Location**

The runtime is either **BYOA** (a custom agent on the customer's BTP subaccount using LangGraph, AG2, CAP, or CrewAI, registering to the SAP-managed Agent Gateway via A2A) or **Joule Studio 2.0** (low-code canvas or CLI). **BYOA is the standard runtime for productization-intent pro-code agents — not a fallback.**

<table>
<caption><b>Runtime Comparison: Joule Studio 2.0 vs. BYOA</b></caption>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top"></th>
<th valign="top">Low-Code Runtime</th>
<th valign="top">Pro-Code Runtime</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">Build surface</td>
<td valign="top">Joule Studio 2.0 (n8n canvas / CLI)</td>
<td valign="top">BYOA (LangGraph / AG2 / CAP / CrewAI on customer BTP subaccount)</td>
</tr>
<tr>
<td valign="top">Registers to</td>
<td valign="top">SMJ managed runtime (automatic)</td>
<td valign="top">SMJ Agent Gateway via A2A</td>
</tr>
<tr>
<td valign="top">When to choose</td>
<td valign="top">Simple skills, citizen development, rapid POC</td>
<td valign="top">Complex or stateful agents with productization intent (the standard FDE case)</td>
</tr>
</tbody>
</table>

**Two FDE delivery types apply:**

-   *IAI CAE* (customer-specific first, productize later) → BYOA agent on the customer subaccount with SMJ orchestrator.
-   *Standard Industries Development* (build into SAP standard) → Joule Studio 2.0 on SMJ.

**Default rule:** the orchestrator is SMJ for all new engagements; choose the runtime (BYOA vs. Joule Studio 2.0) by complexity and productization intent. Use a **CMJ orchestrator** only for a concrete, documented reason — sovereign region where SMJ is unavailable, non-standard Kyma required by the customer's infrastructure team, or an engagement already in progress on CMJ. Document the reason in Discovery and in the productization package.


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_phased_ga"/>

## Phased GA — Honest Status (November 2026)

GA is not binary — it is layered. Do not represent the Joule Studio 2.0 custom-agent managed runtime as broadly generally available without the qualification below.

<table>
<caption><b>GA Status by Layer</b></caption>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top">Layer</th>
<th valign="top">Status</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">SMJ / THE BOX orchestrator</td>
<td valign="top"><b>GA since 2026-06-01</b></td>
</tr>
<tr>
<td valign="top">Joule Studio 2.0 build environment</td>
<td valign="top"><b>Phased GA</b> (fair-use limits apply)</td>
</tr>
<tr>
<td valign="top">Custom-agent managed runtime on SMJ</td>
<td valign="top"><b>EAC-gated / select customers only</b>, free through December 2026 — not broadly GA</td>
</tr>
<tr>
<td valign="top">Outbound A2A (agent registered and surfaced in Joule)</td>
<td valign="top">Available, limited / unidirectional</td>
</tr>
<tr>
<td valign="top">Bidirectional A2A for the managed-runtime BYOA path (Joule ↔ custom agent)</td>
<td valign="top"><b>Transitional — not GA</b> (SAP Architecture Center "transitional state")</td>
</tr>
<tr>
<td valign="top">Inbound / external-to-SAP A2A (third party invokes an SAP agent via ARD / Agent Gateway Consumer)</td>
<td valign="top"><b>Transitional — not GA</b> (Architecture Center); intended future mechanism; <b>H1 2027 target</b></td>
</tr>
</tbody>
</table>

**Practical rule for engagements starting now:** architect the **BYOA + outbound-A2A** path (functional today); treat full Joule Studio managed-runtime deployment and bidirectional A2A as an **H1 2027 target**.


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_js20_runtime"/>

## The Joule Studio 2.0 (Low-Code) Runtime on SMJ

The following workflow applies when the custom-agent runtime is Joule Studio 2.0 on the SMJ orchestrator:

1.  **Build** the agent in Joule Studio 2.0 — canvas (n8n), Joule Studio CLI (`jl`), or the A2A Agent Toolkit for pro-code components.
2.  **Deploy** via Joule Studio — the managed runtime handles BTP provisioning, authentication, and the observability harness automatically.
3.  **Integrate with Joule Work** — automatic; no manual capability bundle or `joule deploy` command is required.
4.  **Govern** via SAP AI Agent Hub — the agent appears in the Agent & MCP Registry automatically; the IT admin validates and activates it.

The following table shows what the **managed runtime** provides automatically, compared to the manual steps required for a BYOA pro-code runtime.

<table>
<caption><b>Managed Runtime: Automatic vs. Manual Wiring</b></caption>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top">Concern</th>
<th valign="top">BYOA Pro-Code (Manual)</th>
<th valign="top">Joule Studio 2.0 Managed Runtime (Automatic)</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">OAuth2 / IAS authentication</td>
<td valign="top">FDE team wires manually</td>
<td valign="top">Automatic</td>
</tr>
<tr>
<td valign="top">OpenTelemetry harness (TR12)</td>
<td valign="top">Manual — OTel to SAP Cloud ALM (or a self-managed collector in sovereign regions)</td>
<td valign="top">Automatic</td>
</tr>
<tr>
<td valign="top">Agent identity + BAIP registration</td>
<td valign="top">Manual capability bundle</td>
<td valign="top">Automatic</td>
</tr>
<tr>
<td valign="top">Joule Work surface integration</td>
<td valign="top">Manual <code>joule deploy</code></td>
<td valign="top">Automatic</td>
</tr>
<tr>
<td valign="top">AI Agent Hub registry entry</td>
<td valign="top">Not automatic</td>
<td valign="top">Automatic (IT admin activates)</td>
</tr>
</tbody>
</table>

> ### Note:  
> Newly deployed Joule Studio 2.0 agents appear in the AI Agent Hub in a "not validated" state until the customer IT admin explicitly activates them. Include this activation step in every productization handover checklist — the agent is deployed and functional, but not in full governance scope until activated.

Joule Studio 2.0 agents run in a **customer-specific SMJ tenant**, not a shared SAP environment. Data residency and sovereign-boundary guarantees are maintained per customer; the "SAP-managed" label refers to the operational infrastructure, not data ownership. Agent telemetry is pushed to SAP Cloud ALM, an SAP-operated service that is region-bound to the customer's contracted data-residency region — not an in-tenant store.


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_byoa_runtime"/>

## The BYOA (Pro-Code) Runtime

### How a BYOA Agent Appears in Joule

From the end user's perspective, a BYOA agent looks identical to a native SAP agent — it appears as a Joule Assistant in the Joule sidebar. The wiring model is invisible to the user.

Under the hood, Joule calls an HTTPS endpoint on the customer's BTP subaccount (Cloud Foundry or Kyma) via the **A2A protocol**. Under the SMJ default, the BYOA agent registers to the SAP-managed Agent Gateway via A2A. In a sovereign or in-progress **CMJ-orchestrator** case, the FDE team instead wires it manually via `joule deploy`.

**A2A direction — note the phased-GA caveat:**

-   **Outbound (Joule → your agent):** available today; the standard BYOA path.
-   **Bidirectional for the managed-runtime BYOA path:** transitional — not GA (see the GA table above). Architect for outbound and async callbacks now.
-   **Inbound / external-to-SAP (third party invokes an SAP agent):** transitional — not GA (SAP Architecture Center). The intended future mechanism is ARD-based discovery, SAP-managed formations (Identity → Trust → Authorization), and Agent Gateway Consumer registration. H1 2027 target.

### What the FDE Team Must Wire Manually

The following items are handled automatically by the Joule Studio 2.0 managed runtime but must be wired manually by the FDE team on the BYOA pro-code runtime.

<table>
<caption><b>Manual Wiring Requirements for BYOA</b></caption>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top">What</th>
<th valign="top">Why It Matters</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top"><b>Auth</b> (OAuth2 CC or IAS)</td>
<td valign="top">Joule must authenticate to call the agent.</td>
</tr>
<tr>
<td valign="top"><b>Public Agent Card endpoint</b></td>
<td valign="top">Joule reads <code>/.well-known/agent.json</code> during registration — must be publicly reachable.</td>
</tr>
<tr>
<td valign="top"><b>A2A registration / capability bundle</b></td>
<td valign="top">Registers the agent to SMJ via A2A or, for a CMJ orchestrator, the four-file <code>joule deploy</code> bundle.</td>
</tr>
<tr>
<td valign="top"><b>Observability</b></td>
<td valign="top">Not automatic — wire OTel to SAP Cloud ALM (or a self-managed OTLP collector in sovereign regions such as NS2 where Cloud ALM is unavailable).</td>
</tr>
<tr>
<td valign="top"><b>Regional orchestrator check</b></td>
<td valign="top">If SMJ is not available in-region, a CMJ orchestrator (separate prerequisite) is required.</td>
</tr>
</tbody>
</table>

### The Three Steps for BYOA Delivery

<table>
<caption><b>BYOA Delivery Steps and Gates</b></caption>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top">Step</th>
<th valign="top">What Happens</th>
<th valign="top">Gate to Pass Before Moving On</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top"><b>1 — Local agent</b></td>
<td valign="top">Scaffold the agent; answer real A2A messages with mock or test data.</td>
<td valign="top"><code>SendMessage</code> returns a valid response locally.</td>
</tr>
<tr>
<td valign="top"><b>2 — Deploy to BTP</b></td>
<td valign="top">Push to Cloud Foundry or Kyma; public HTTPS endpoint is live.</td>
<td valign="top">Agent Card reachable at the public URL; smoke test passes over HTTPS.</td>
</tr>
<tr>
<td valign="top"><b>3 — Register to Joule</b></td>
<td valign="top">Register to SMJ Agent Gateway via A2A (or <code>joule deploy</code> for a CMJ orchestrator); test from the Joule sidebar.</td>
<td valign="top">Capability shows ACTIVE; Joule routes a real prompt to the agent.</td>
</tr>
</tbody>
</table>

> ### Note:  
> **60-second constraint (both runtimes):** Joule expects a synchronous response within 60 seconds. Agent workflows involving chained LLM calls or external data fetches must fit within this window or use the async A2A pattern (push notifications). Measure p99 latency under realistic load before assuming synchronous delivery is viable.


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_ownership"/>

## Ownership Summary

<table>
<caption><b>Ownership by Runtime</b></caption>
<colgroup>
<col/>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top"></th>
<th valign="top">Joule Studio 2.0 Runtime (on SMJ)</th>
<th valign="top">BYOA Pro-Code Runtime</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">Writes the agent</td>
<td valign="top">FDE team</td>
<td valign="top">FDE team</td>
</tr>
<tr>
<td valign="top">Manages the Joule / BAIP account</td>
<td valign="top">SAP (customer subscribes to SMJ)</td>
<td valign="top">SAP (SMJ); customer IT only if a CMJ orchestrator is used</td>
</tr>
<tr>
<td valign="top">Wires auth, observability, Joule registration</td>
<td valign="top"><b>Automatic (managed runtime)</b></td>
<td valign="top">FDE team (manual)</td>
</tr>
<tr>
<td valign="top">Operates post-handover</td>
<td valign="top">Customer IT + CIS T&amp;O (via supervisor mechanism)</td>
<td valign="top">Customer IT + CIS T&amp;O (via supervisor mechanism)</td>
</tr>
<tr>
<td valign="top">IP &amp; productization</td>
<td valign="top">SAP — standard FDE model, no change</td>
<td valign="top">SAP — standard FDE model, no change</td>
</tr>
</tbody>
</table>


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_deployment_landscape"/>

## Current Deployment Landscape

The following diagram illustrates the two axes and the relationship between the orchestrator and the runtime options.

```
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


<a name="loioa0231c5e241a4bb88a953b3fae10b9cc__section_status"/>

## Status and Open Items

<table>
<caption><b>Feature Status and Open Items</b></caption>
<colgroup>
<col/>
<col/>
</colgroup>
<thead>
<tr>
<th valign="top">Item</th>
<th valign="top">Status</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">SMJ orchestrator</td>
<td valign="top"><b>GA since 2026-06-01</b> — default for all new engagements</td>
</tr>
<tr>
<td valign="top">Joule Studio 2.0 build environment</td>
<td valign="top"><b>Phased GA</b> (fair-use limits)</td>
</tr>
<tr>
<td valign="top">JS2.0 custom-agent managed runtime</td>
<td valign="top"><b>EAC-gated / select customers</b>, free through December 2026 — not broadly GA; full managed-runtime deployment is an H1 2027 target</td>
</tr>
<tr>
<td valign="top">CMJ (customer-managed orchestrator)</td>
<td valign="top"><b>Deprecated</b> — migration target; use only where SMJ is unavailable in-region or for in-progress engagements; document the reason</td>
</tr>
<tr>
<td valign="top">BYOA (pro-code runtime)</td>
<td valign="top"><b>Standard</b> runtime for productization-intent agents — registers to SMJ via A2A; not a fallback</td>
</tr>
<tr>
<td valign="top">Bidirectional A2A (managed-runtime BYOA path)</td>
<td valign="top"><b>Transitional — not GA</b> (Architecture Center); architect for outbound and async callbacks</td>
</tr>
<tr>
<td valign="top">Inbound / external-to-SAP A2A (ARD / Agent Gateway Consumer)</td>
<td valign="top"><b>Transitional — not GA</b> (Architecture Center); intended future mechanism; <b>H1 2027 target</b></td>
</tr>
<tr>
<td valign="top">Formal IAI/CIS guidance on delivery path</td>
<td valign="top">Effective guidance: SMJ orchestrator default; BYOA or Joule Studio 2.0 runtime by complexity; CMJ only when specifically needed. Monitor the IAI/CIS alignment channel for formal policy confirmation.</td>
</tr>
</tbody>
</table>

**Related Information**  


[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)

[Plan and Build AI Agents](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/e2a29b0c422e4aceadee658f4bbab71b.html)
