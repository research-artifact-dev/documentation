<!-- loioca0bf9b3a07742c4a051fd62ebea4c19 -->

# Context Layer Overview

The **Context Layer** is a CIS Technology & Operations initiative that transforms CIS delivery knowledge into a queryable **Knowledge Graph** on SAP Business Technology Platform, integrated into SAP's Enterprise Knowledge Network (EKN/EKX). Where the Agent Development concept addresses how to build, govern, and run agents, the Context Layer answers what CIS has already built and what can be reused across engagements.


<a name="loioca0bf9b3a07742c4a051fd62ebea4c19__section_architecture"/>

## Architecture

The Context Layer follows a **multi-source ingestion pipeline**: multiple source systems feed into AI Core embeddings and semantic clustering, followed by an EARL BAD/SAD process-mapping stage, before being loaded into a CIS-specific, access-controlled **CISEKN endpoint** — separate from the shared SAP Enterprise Knowledge Graph. The result is queryable via **Joule Work** (primary front end, via MCP Connector) or directly through EKX. Data joining happens in the graph at build time, not at query time, making the Knowledge Graph the authoritative integration layer.

The **two-graph model** (CISEKN separate from the shared Enterprise Knowledge Graph) enforces access control: CIS proprietary delivery knowledge never merges into the shared enterprise graph. CIS architects and FDE teams access it exclusively through the access-controlled CIS endpoint.

> ### Note:  
> The current working prototype is **CIS Capability Scout v2.0.0**, deployed on BTP CF eu12. It exposes 7 tools across 3 data sources — 96 real CIS projects from January 2025 to October 2026, a capability catalog, and EKN live data — with end-to-end confirmation via Joule Work. The productive CISEKN endpoint is pending a self-service BTP subscription to Knowledge Graph Foundation once Content Infrastructure registers the CISEKN source. The delivery horizon is February 2027, with a Vision Backlog of Jira-ready epics (EP-1–EP-13).


<a name="loioca0bf9b3a07742c4a051fd62ebea4c19__section_source_systems"/>

## Source Systems

The Context Layer has evaluated multiple project tracking systems as Knowledge Graph inputs. **Cortex** (v0.1.0, BTP CF eu12) is identified as the richest source — a purpose-built CIS/FDE engagement tracker with structured `problem_statement`, `solution_summary`, `ai_patterns`, and `iai_stack_layers` fields covering the full 5-phase lifecycle. The CIS Project Tracker covers delivery phases only, while the MXP IAI Tool fills the pre-project qualification gap. SAP Cloud ALM is a Phase 2 candidate for solution-level intelligence.

<table>
<caption><b>Source Systems and Knowledge Graph Relevance</b></caption>
<colgroup>
<col style="width: 20%"/>
<col style="width: 20%"/>
<col style="width: 35%"/>
<col style="width: 25%"/>
</colgroup>
<thead>
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
</thead>
<tbody>
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

REST API confirmed; data volume to confirm

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

API in development (~Dec 2026)

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

CIS Git Repos

</td>
<td valign="top">

Phase 4–5 artifacts

</td>
<td valign="top">

Phase 2 — direct access, not KG-ingested

</td>
<td valign="top">

GitHub MCP PoC complete (Oct 2026)

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

Phase 3 — evaluated for Delivery Intelligence

</td>
<td valign="top">

Evaluated

</td>
</tr>
</tbody>
</table>


<a name="loioca0bf9b3a07742c4a051fd62ebea4c19__section_touchpoints"/>

## Touchpoints with the Agent Development Concept

The Context Layer intersects with several topics in the Agent Development concept. Each touchpoint describes a concrete dependency or integration point.

**Token Efficiency — L1 Context Loading**

The CIS Knowledge Graph as MCP server is a direct implementation of the **L1 context-loading** mechanism described in [Token Efficiency](token-efficiency-a49034e.md). Instead of bulk-loading CIS project context into every agent session, an FDE engineer's agent queries the Knowledge Graph on demand — pulling only the capability patterns relevant to the current engagement phase. This reduces AI Unit cost per session and is the most immediate efficiency win once the productive CISEKN endpoint is live. The MCP server belongs in the `apm.yml` of the CIS Starter Kit. With Cortex as the primary data source, query results include structured AI patterns and technology stack fields, enabling finer-grained L1 context selection than free-text descriptions alone.

**Reusable Components and the CIS Starter Kit**

The Context Layer is the runtime data source behind capability reuse. [Reusable Components](reusable-components-b03a699.md) defines the three-layer reusability stack (deploy as-is / parameterize / engagement-specific) — the Knowledge Graph is what makes the "deploy as-is" tier **discoverable**. An FDE architect can query *"has CIS built an Order-to-Cash agent for automotive?"* before deciding which reuse tier applies. The MCP server is a Starter Kit component. Cortex's `ai_patterns`, `sap_technologies`, and `iai_stack_layers` fields make these queries structurally answerable once Cortex data is ingested.

**CIS FDE Project Scaffold — Shared Dependency**

Both the Context Layer and the Agent Development project depend on a standard that does not yet exist: a consistent CIS FDE project structure with defined file conventions and metadata. The Context Layer requires it to parse delivery repositories for capability patterns (Phase 2). The Agent Development project requires it to attribute AI Unit costs to specific engagements (see [Token Efficiency](token-efficiency-a49034e.md)). This T&O-owned workstream gates both initiatives.

**Agent Evaluation — Online Observability**

Once CIS-built agents query the EKN Knowledge Graph at runtime, those query calls appear in MLflow production traces. The Context Layer becomes observable through the same **`aeval` online evaluation pipeline** described in [Agent Evaluation](agent-evaluation-80dcfa4.md) — KG query cost, frequency by engagement, and query patterns are all signals for evaluating agent grounding quality.

**FDE Discovery Phase**

The most immediate human-facing value is in the Discovery sub-phases (Define and Prioritize): an FDE architect can query the CIS Knowledge Graph conversationally — *"what did CIS deliver for similar customers in this process area?"* — before scoping the engagement. This reduces the risk of scoping work CIS has already completed. Cortex's structured coverage of Phases 1–3 (qualification → discovery → value proof) means the Knowledge Graph will hold context from the earliest engagement phases, giving architects pre-delivery pattern signals rather than post-delivery evidence only.


<a name="loioca0bf9b3a07742c4a051fd62ebea4c19__section_north_star"/>

## Context Layer in the North Star Architecture

The Context Layer is the **CIS-specific extension of the Foundation Layer's context engineering capability** — adding a CIS domain graph alongside the SAP Knowledge Graph that EKX already exposes. BAIP's Context sub-layer (Knowledge Graph, 7M+ fields) covers SAP's product and process knowledge; the Context Layer fills the gap with CIS engagement and capability knowledge.

Key architectural principles:

-   The **CISEKN endpoint** is separate from the shared Enterprise Knowledge Graph to preserve access control over CIS proprietary delivery knowledge.
-   The Knowledge Graph is the integration layer; all data joining happens at build time, not at query time, ensuring consistent and efficient query responses.
-   **Joule Work** (via MCP Connector) is the primary front end, with EKX as an alternative query interface.
-   The Vision Backlog (EP-1–EP-13) defines the phased delivery path toward a fully productive CISEKN endpoint, with a February 2027 horizon.

> ### Note:  
> The source repository for the Context Layer is maintained at `https://github.tools.sap/CIS-Technology-Operations/context_layer`. Local path: `C:\CIS\context_layer`.

**Related Information**  


[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html)

[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)
