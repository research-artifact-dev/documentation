<!-- loiocb821e538206470e99c33cad81331c22 -->

# Context Layer Overview

The CIS Context Layer is a CIS T&O initiative that turns CIS delivery knowledge into a queryable Knowledge Graph (KG) on SAP BTP, integrated into SAP's Enterprise Knowledge Network (EKN/EKX). Where the agent development concept answers *how to build, govern, and run agents*, the Context Layer answers *what has CIS already built, and what can be reused.*


<a name="loiocb821e538206470e99c33cad81331c22__section_architecture"/>

## Architecture

The Context Layer follows a multi-source pipeline architecture: multiple source systems → AI Core embeddings + semantic clustering → EARL BAD/SAD process mapping → KG loaded into a **CIS-specific, access-controlled CISEKN endpoint** (separate from the shared SAP Enterprise Knowledge Graph) → queryable via **Joule Work** (primary front end, via MCP Connector) or EKX.

The KG is the integration layer; data joining happens in the graph at build time, not at query time.

**Current state (November 2026)**: Working prototype **CIS Capability Scout v2.0.0** deployed on BTP CF eu12 — 7 tools across 3 data sources (96 real CIS projects Jan 2025–Oct 2026, capability catalog, EKN live), confirmed end-to-end with Joule Work. Productive CISEKN endpoint pending: self-service BTP subscription to Knowledge Graph Foundation once Content Infrastructure registers the CISEKN source. Vision Backlog with Jira-ready epics (EP-1–EP-13) documented; Feb 2027 delivery horizon.

**Repository**: `C:\CIS\context_layer` (also at `https://github.tools.sap/CIS-Technology-Operations/context_layer`)


<a name="loiocb821e538206470e99c33cad81331c22__section_source_systems"/>

## Source Systems

The Context Layer has evaluated multiple project tracking systems as KG inputs. **Cortex** (v0.1.0, BTP CF eu12) is identified as the richest source — purpose-built CIS/FDE engagement tracker with structured `problem_statement`, `solution_summary`, `ai_patterns`, and `iai_stack_layers` fields covering the full 5-phase lifecycle. CIS Project Tracker covers delivery phases only; MXP IAI Tool fills the pre-project qualification gap. SAP Cloud ALM is a Phase 2 candidate for solution-level intelligence.

**Table: Source System Evaluation**

<table>
<tr>
<th valign="top">

Source

</th>
<th valign="top">

Lifecycle Coverage

</th>
<th valign="top">

KG Relevance

</th>
<th valign="top">

Status

</th>
</tr>
<tr>
<td valign="top">

**Cortex**

</td>
<td valign="top">

Phase 1–5 (full)

</td>
<td valign="top">

**Highest** — richest structured use-case data; primary Phase 1+2 candidate

</td>
<td valign="top">

REST API confirmed; data volume to confirm (Ajay Tikare)

</td>
</tr>
<tr>
<td valign="top">

CIS Project Tracker

</td>
<td valign="top">

Phase 4–5 (delivery+)

</td>
<td valign="top">

High — current anchor source; 96 projects in prototype

</td>
<td valign="top">

API in development (~Dec 2026, Philipp Koehler)

</td>
</tr>
<tr>
<td valign="top">

MXP IAI Tool

</td>
<td valign="top">

Phase 1–2 (pre-project)

</td>
<td valign="top">

High — fills pre-project qualification gap; `mxp_opportunity_id` links to Cortex

</td>
<td valign="top">

MCP + Joule Skill live; data model TBD

</td>
</tr>
<tr>
<td valign="top">

CIS Git repos

</td>
<td valign="top">

Phase 4–5 artifacts

</td>
<td valign="top">

Phase 2 — direct access, not KG-ingested

</td>
<td valign="top">

GitHub MCP PoC complete (CTXL-67, Oct 2026)

</td>
</tr>
<tr>
<td valign="top">

SAP Cloud ALM

</td>
<td valign="top">

Phase 4 (what was built)

</td>
<td valign="top">

Phase 2 — requirements, scoping, test coverage per project

</td>
<td valign="top">

Identified; EKG entity confirmed; access TBD

</td>
</tr>
<tr>
<td valign="top">

EPPM, FELIPE, PEP

</td>
<td valign="top">

Delivery structure, financials

</td>
<td valign="top">

Phase 3

</td>
<td valign="top">

Evaluated; for Delivery Intelligence (EP-14)

</td>
</tr>
</table>


<a name="loiocb821e538206470e99c33cad81331c22__section_touchpoints"/>

## Touchpoints with the Agent Development Concept

### Token Efficiency — L1 Context Loading

The CIS KG as MCP server is a direct implementation of the L1 mechanism described in [Token Efficiency](token-efficiency-8449831.md): instead of bulk-loading CIS project context into every agent session, an FDE engineer's agent queries the KG on demand — pulling only the capability patterns relevant to the current engagement phase. This reduces AI Unit cost per session and is the most immediate efficiency win once the productive CISEKN endpoint is live.

The MCP server belongs in `apm.yml` of the CIS Starter Kit (see [Reusable Components](reusable-components-ff99fe9.md)). With Cortex as the primary data source, query results include structured AI patterns and technology stack fields — enabling finer-grained L1 context selection than free-text descriptions alone.

### Reusable Components and CIS Starter Kit

The Context Layer is the runtime data source behind capability reuse. [Reusable Components](reusable-components-ff99fe9.md) defines the three-layer reusability stack (deploy as-is / parameterize / engagement-specific) — the KG is what makes the "deploy as-is" tier discoverable. An FDE architect can query *"has CIS built an Order-to-Cash agent for automotive?"* before deciding which tier applies.

The MCP server is a Starter Kit component. Cortex's `ai_patterns`, `sap_technologies`, and `iai_stack_layers` fields make these queries structurally answerable once Cortex data is ingested into the KG.

### CIS FDE Project Scaffold — Shared Dependency

Both the Context Layer and the [Token Efficiency](token-efficiency-8449831.md) cost attribution work depend on a standard that does not yet exist: a consistent CIS FDE project structure with defined file conventions and metadata. The Context Layer needs it to parse delivery repos for capability patterns (Phase 2). The Agent Development concept needs it to attribute AI Unit costs to specific engagements. This is a T&O-owned workstream that gates both.

### Agent Evaluation — Online Observability

Once CIS-built agents query the EKN KG at runtime, those KG query calls appear in MLflow production traces. The Context Layer becomes observable through the same `aeval` online evaluation pipeline described in [Agent Evaluation](agent-evaluation-bb08bc0.md) — KG query cost, frequency by engagement, and pattern of queries are all signals for evaluating agent grounding quality.

### FDE Discovery Phase — Define and Prioritize

The most immediate human-facing value: during Discovery 3 (Define) and Discovery 4 (Prioritize), an FDE architect can query the CIS KG conversationally — *"what did CIS deliver for similar customers in this process area?"* — before scoping the engagement. This reduces the risk of scoping work that CIS has already done.

Cortex's structured coverage of Phases 1–3 (qualification → discovery → value proof) means the KG will eventually hold context from the earliest engagement phases, giving architects pre-delivery pattern signals, not just post-delivery delivery evidence.


<a name="loiocb821e538206470e99c33cad81331c22__section_north_star"/>

## Context Layer in the North Star Architecture

The Context Layer is the **CIS-specific extension of the Foundation Layer's context engineering capability** — adding a CIS domain graph alongside the SAP Knowledge Graph that EKX already exposes.

BAIP's Context sub-layer (Knowledge Graph, 7M+ fields) covers SAP's product and process knowledge; the Context Layer fills the gap with CIS engagement and capability knowledge. The **two-graph model** (CISEKN separate from the shared EKG) enforces access control: CIS proprietary delivery knowledge never merges into the shared enterprise graph — CIS architects and FDE teams access it through the access-controlled CIS endpoint only.

**Related Information**  


[Token Efficiency](token-efficiency-8449831.md)

[Reusable Components](reusable-components-ff99fe9.md)

[Agent Evaluation](agent-evaluation-bb08bc0.md)

[Introduction](introduction-cc824ed.md)

[SAP AI Core](https://help.sap.com/docs/AI_CORE/2d6c5984063c40a59eda62f4a9135bee/d029a32c22fb45fbb607e6a2c48c8a0e.html "SAP AI Core — the runtime for AI scenarios including model training, inference, and agent execution on SAP BTP.") :arrow_upper_right:

[SAP Integration Suite MCP Server](https://help.sap.com/docs/SAP_INTEGRATION_SUITE/9519789d5664487f8b9cd89eba514477/94957bc87247462ebdfd65e72512122a.html "SAP Integration Suite MCP Server — connect MCP-compatible agents to SAP Integration Suite capabilities.") :arrow_upper_right:

[Joule Work – Develop](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/9d86b3b7b9534915a0f542ea7ea9f839.html "Joule Work Develop — Joule Studio in the Joule Work experience for building and managing AI agent workflows.") :arrow_upper_right:
