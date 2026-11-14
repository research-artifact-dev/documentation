<!-- loiocc824ede03f04c40a2de1624834cb769 -->

# Introduction: Why Agent Governance Matters for Industry AI

SAP's Industry AI portfolio brings autonomous agents to the most complex, high-stakes processes in industrial enterprises — batch release decisions in regulated manufacturing, commodity hedging, pricing at scale, and asset maintenance strategies. Governing these agents is not an optional add-on; it is the condition under which enterprises can trust agents to act autonomously on high-stakes decisions.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_opportunity_and_risk"/>

## The Opportunity and the Risk

SAP's Industry AI portfolio brings autonomous agents to processes where getting it right generates significant business value — and where getting it wrong has regulatory, financial, or operational consequences.

This is precisely what makes AI agents different from conventional software. A traditional application either works or it doesn't; the failure mode is visible. An AI agent reasons, selects tools, and acts on its own judgment. It can appear to work — completing each individual step correctly — while still producing wrong outputs, following unsafe paths, or drifting in quality over time in ways that are not immediately visible.

Governing agents well is the foundation on which value-based contracts (10–30% of yearly customer benefit) can be justified.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_what_this_concept_addresses"/>

## What This Concept Addresses

This document was developed by the CIS (Customer Industry Solutions) Technology and Operations team as a design concept for building, monitoring, and governing AI agents within SAP's Industry AI and FDE (Forward Deployed Engineering) context. SAP positions Industry AI as a fifth element of the Autonomous Enterprise stack — providing "built-in vertical expertise with sector-specific process logic, data models, and regulatory logic" on top of the generic agentic platform. CIS is the delivery engine for this layer: the central, customer-anchored Industry AI acceleration engine that instantiates these domain-specific capabilities with and for strategic customers.

This concept is a reference architecture and design pattern collection intended for use in:

-   **FDE engagements**: as a design reference during Discovery and Development phases
-   **Customer conversations**: as a framework for discussing agent governance with enterprise architects and process owners
-   **Internal enablement**: as a shared language for CIS practitioners designing Industry AI solutions

The concept answers the **core design questions** that arise in every high-stakes agent deployment:

**Table: Core Design Questions and Topic References**

<table>
<tr>
<th valign="top">

Design Question

</th>
<th valign="top">

Topic

</th>
<th valign="top">

Key Sections

</th>
</tr>
<tr>
<td valign="top">

How do we ensure an agent works correctly — at build time and at runtime?

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

Three evaluation dimensions; build-time quality; runtime quality; iteration signals

</td>
</tr>
<tr>
<td valign="top">

How do we handle failures autonomously: self-correct vs. escalate?

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md), [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Escalation path; supervisor functions; escalation thresholds

</td>
</tr>
<tr>
<td valign="top">

Does a supervisor agent pattern make sense within SAP's architecture?

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

Supervision tier; alternatives; three functions; architecture

</td>
</tr>
<tr>
<td valign="top">

How do we protect customer data while still enabling operational monitoring?

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Security and Data Protection](security-and-data-protection-a403094.md)

</td>
<td valign="top">

Data protection design; data protection architecture; sovereign AI; legal alignment

</td>
</tr>
<tr>
<td valign="top">

How do we address security at development time AND at runtime?

</td>
<td valign="top">

[Security and Data Protection](security-and-data-protection-a403094.md)

</td>
<td valign="top">

Development-time security; runtime security; agent identity; prompt injection

</td>
</tr>
<tr>
<td valign="top">

How do we enforce precision guardrails for business-critical numerical outputs?

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

Guardrail stack; four layers; confidence and uncertainty

</td>
</tr>
<tr>
<td valign="top">

How do we manage agent lifecycle after productization: monitor, maintain, and continuously improve?

</td>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

</td>
<td valign="top">

Lifecycle shift; supervisor as CIS proxy; operating rhythm; continuous improvement loop

</td>
</tr>
<tr>
<td valign="top">

When does an Industry AI use case require multiple specialist agents, and how do we orchestrate and govern them safely?

</td>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
<td valign="top">

When multi-agent; SAP orchestration model; three topologies; quality extension; supervisor extension

</td>
</tr>
<tr>
<td valign="top">

How do we version, roll back, and migrate agents across environments without service disruption?

</td>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Versioning strategy; rollback mechanics; environment migration

</td>
</tr>
<tr>
<td valign="top">

Which design components can be reused across engagements, and what constitutes a CIS Starter Kit for efficient delivery at scale?

</td>
<td valign="top">

[Reusable Components](reusable-components-ff99fe9.md)

</td>
<td valign="top">

Three-layer reusability stack; component-by-component analysis; CIS Starter Kit

</td>
</tr>
<tr>
<td valign="top">

How do we manage token efficiency across FDE build-time and Industry AI runtime to maintain a sustainable cost per correct outcome?

</td>
<td valign="top">

[Token Efficiency](token-efficiency-8449831.md)

</td>
<td valign="top">

L0–L6 framework; build-time guidelines; runtime efficiency; quality–efficiency connection

</td>
</tr>
<tr>
<td valign="top">

How do we evaluate agent quality systematically across an FDE sprint — from first prototype through to production monitoring?

</td>
<td valign="top">

[Agent Evaluation](agent-evaluation-bb08bc0.md)

</td>
<td valign="top">

Test case DSL; Quality Flywheel; online evaluation in production; evaluation dimensions checklist

</td>
</tr>
<tr>
<td valign="top">

How do we ensure customer organizations are actually able to adopt and sustain the agent after the FDE team departs?

</td>
<td valign="top">

[Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md)

</td>
<td valign="top">

Adoption Strategy lifecycle; gradual release (4 stages); role-based training (7 audiences); enablement readiness checklist

</td>
</tr>
</table>


<a name="loiocc824ede03f04c40a2de1624834cb769__section_north_star_architecture"/>

## North Star Architecture

The **North Star Architecture** defines the standard component stack for CIS Industry AI agent deployments on SAP BTP. It establishes the governance and runtime boundaries that every FDE engagement inherits — from the Joule Studio build environment through the BAIP Managed Runtime, down to the MCP-connected systems of record.

```mermaid
graph TB
    FDE["FDE Engineer (Joule Work)"] --> JouleStudio["Joule Studio\n(Intent → Deploy)"]
    JouleStudio --> BAIP["BAIP Managed Runtime\n(SAP BTP)"]
    BAIP --> Supervisor["Supervisor Agent\n(Governance Layer)"]
    Supervisor --> WorkerA["Worker Agent A\n(Domain Specialist)"]
    Supervisor --> WorkerB["Worker Agent B\n(Domain Specialist)"]
    Supervisor --> WorkerC["Worker Agent C\n(Domain Specialist)"]
    WorkerA --> AICore["SAP AI Core\nGenAI Hub"]
    WorkerB --> AICore
    WorkerC --> AICore
    WorkerA --> MCP["MCP Servers\n(SAP Integration Suite)"]
    WorkerB --> MCP
    MCP --> SoR["Systems of Record\n(S/4HANA, ECC, ...)"]
    Supervisor --> EKX["SAP Knowledge Graph\n(EKX)"]
    Supervisor --> CISEKN["CIS Context Layer\n(CISEKN)"]
    Supervisor --> TaskCenter["SAP Task Center\n(Human Escalation)"]
    BAIP --> MLflow["MLflow\n(Observability)"]
```

**Table: North Star Architecture — Key Components**

<table>
<tr>
<th valign="top">

Component

</th>
<th valign="top">

Role in the Architecture

</th>
<th valign="top">

SAP Product or Service

</th>
</tr>
<tr>
<td valign="top">

Joule Studio

</td>
<td valign="top">

Intent-based development environment — translates a business requirement into a deployed agent solution without requiring low-level infrastructure work.

</td>
<td valign="top">

[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio is SAP's AI-first environment to create enterprise solutions from intent to deployment.") :arrow_upper_right:

</td>
</tr>
<tr>
<td valign="top">

BAIP Managed Runtime

</td>
<td valign="top">

The governed cloud execution environment on SAP BTP where deployed agents run. Enforces deployment boundaries, access policies, and observability.

</td>
<td valign="top">

SAP Business AI Platform (BAIP) on SAP BTP

</td>
</tr>
<tr>
<td valign="top">

Supervisor Agent

</td>
<td valign="top">

Orchestration and governance tier. Routes tasks to worker agents, validates outputs before committing to systems of record, and escalates to humans when confidence thresholds are not met.

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
</tr>
<tr>
<td valign="top">

Worker Agents

</td>
<td valign="top">

Domain-specialist agents that execute specific reasoning or data-retrieval tasks. Each holds minimal scope — only the MCP tools required for its function.

</td>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
</tr>
<tr>
<td valign="top">

SAP AI Core / GenAI Hub

</td>
<td valign="top">

Foundation model runtime. Provides governed access to LLMs and embedding models within enterprise data-residency and compliance constraints.

</td>
<td valign="top">

[SAP AI Core](https://help.sap.com/docs/AI_CORE/2d6c5984063c40a59eda62f4a9135bee/d029a32c22fb45fbb607e6a2c48c8a0e.html "SAP AI Core is the runtime and lifecycle management foundation for AI scenarios on SAP BTP.") :arrow_upper_right:

</td>
</tr>
<tr>
<td valign="top">

MCP Servers (SAP Integration Suite)

</td>
<td valign="top">

Expose enterprise APIs and data sources to agents in a structured, governed way. Workers call MCP tools rather than hitting systems of record directly.

</td>
<td valign="top">

[SAP Integration Suite — MCP Server Artifacts](https://help.sap.com/docs/SAP_INTEGRATION_SUITE/9519789d5664487f8b9cd89eba514477/94957bc87247462ebdfd65e72512122a.html "SAP Integration Suite enables you to design and manage API artifacts and MCP Server artifacts as part of your integration landscape.") :arrow_upper_right:

</td>
</tr>
<tr>
<td valign="top">

SAP Knowledge Graph (EKX)

</td>
<td valign="top">

SAP's enterprise knowledge graph — 7M+ fields covering SAP products, processes, and business objects. Used for L1 context loading in agent sessions.

</td>
<td valign="top">

SAP Enterprise Knowledge Network (EKN / EKX) on SAP BTP

</td>
</tr>
<tr>
<td valign="top">

CIS Context Layer (CISEKN)

</td>
<td valign="top">

CIS-specific knowledge graph endpoint — queryable index of CIS delivery knowledge (96+ projects, capability patterns, reuse candidates). Access-controlled and separate from the shared EKX graph.

</td>
<td valign="top">

[Context Layer Overview](context-layer-overview-cb821e5.md)

</td>
</tr>
<tr>
<td valign="top">

MLflow Observability

</td>
<td valign="top">

Production trace store. Every agent run writes structured traces; the online evaluation pipeline (`aeval`) reads these traces to compute quality metrics and trigger alerts.

</td>
<td valign="top">

[Agent Evaluation](agent-evaluation-bb08bc0.md)

</td>
</tr>
<tr>
<td valign="top">

SAP Task Center

</td>
<td valign="top">

Human escalation endpoint. When a supervisor agent cannot meet confidence thresholds, it creates a structured task in SAP Task Center for a human decision-maker.

</td>
<td valign="top">

SAP Task Center on SAP BTP

</td>
</tr>
</table>


<a name="loiocc824ede03f04c40a2de1624834cb769__section_how_to_read"/>

## How to Read This Document

Each topic is self-contained but builds on the previous. [Agent Quality Framework](agent-quality-framework-5f15077.md), [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Security and Data Protection](security-and-data-protection-a403094.md), and [Precision Guardrails](precision-guardrails-ae75c89.md) cover design concepts; [Joule Studio Build Map](joule-studio-build-map-67ad030.md) maps all concepts to concrete Joule Studio implementation choices; [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md), [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md), and [Production Operations](production-operations-e24808a.md) address the operational and organizational layers.

**Table: Topics by Content and Audience**

<table>
<tr>
<th valign="top">

Topic

</th>
<th valign="top">

What It Covers

</th>
<th valign="top">

Primary Audience

</th>
</tr>
<tr>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-5f15077.md)

</td>
<td valign="top">

How to measure agent quality across three dimensions (technical, process KPI, business outcomes); build-time and runtime quality; the escalation path

</td>
<td valign="top">

All

</td>
</tr>
<tr>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md)

</td>
<td valign="top">

When and how to deploy a supervisor agent; three supervisor functions; worker-supervisor communication; data protection design

</td>
<td valign="top">

FDE architects, BAIP developers

</td>
</tr>
<tr>
<td valign="top">

[Security and Data Protection](security-and-data-protection-a403094.md)

</td>
<td valign="top">

Agent-specific security threats; the three-tier defense model; agent identity and least-privilege; data flows and legal alignment

</td>
<td valign="top">

FDE architects, security reviewers

</td>
</tr>
<tr>
<td valign="top">

[Precision Guardrails](precision-guardrails-ae75c89.md)

</td>
<td valign="top">

The four-layer guardrail stack for numerical outputs; domain examples; human approval thresholds; audit trail requirements

</td>
<td valign="top">

FDE architects, process owners, compliance

</td>
</tr>
<tr>
<td valign="top">

[Joule Studio Build Map](joule-studio-build-map-67ad030.md)

</td>
<td valign="top">

Concrete implementation of all design concepts using Joule Studio low-code nodes; master build map table; FDE build sequence

</td>
<td valign="top">

FDE application engineers, dev architects

</td>
</tr>
<tr>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md)

</td>
<td valign="top">

Lifecycle management after productization; supervisor as autonomous CIS proxy; operating rhythm, severity tiers, and continuous improvement loop; CoE maturity levels

</td>
<td valign="top">

FDE PMs, CIS T&O leads, customer IT admins

</td>
</tr>
<tr>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md)

</td>
<td valign="top">

When and how to compose multiple specialist agents; three orchestration topologies (flat, per-worker supervised, shared supervisor); supervisor patterns at scale

</td>
<td valign="top">

FDE architects, dev architects

</td>
</tr>
<tr>
<td valign="top">

[Production Operations](production-operations-e24808a.md)

</td>
<td valign="top">

Agent versioning and rollback (five-component version stack); cost and token monitoring; AI Units billing model; environment migration (dev → test → prod); FDE delivery constraints

</td>
<td valign="top">

FDE dev architects, CIS T&O leads

</td>
</tr>
<tr>
<td valign="top">

[Reusable Components](reusable-components-ff99fe9.md)

</td>
<td valign="top">

Three-layer reusability stack; component-by-component analysis; CIS Starter Kit definition; APM per-project setup; Discovery parameterization guide

</td>
<td valign="top">

FDE architects, CIS T&O leads

</td>
</tr>
<tr>
<td valign="top">

[Token Efficiency](token-efficiency-8449831.md)

</td>
<td valign="top">

Two cost pools (build-time FDE engineers vs. runtime deployed agents); SAP L0–L6 optimization framework; build-time engineer guidelines; runtime efficiency; quality–efficiency connection

</td>
<td valign="top">

FDE dev architects, FDE Application Engineers, CIS T&O leads

</td>
</tr>
<tr>
<td valign="top">

[Agent Evaluation](agent-evaluation-bb08bc0.md)

</td>
<td valign="top">

Testing vs. evaluation distinction; SAP aeval framework; test case DSL as a sprint deliverable; Quality Flywheel mapped to sprint weeks 1–12; success rate as primary release gate; evaluation dimensions checklist

</td>
<td valign="top">

All — FDE Application Engineers and Dev Architects especially

</td>
</tr>
<tr>
<td valign="top">

[Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md)

</td>
<td valign="top">

The Enable track as a mandatory parallel workstream; Adoption Strategy as formal handover artifact (7 sections, 9 governance roles); four-stage gradual release; role-based training for 7 audiences; enablement readiness checklist; CoE post-handover support model

</td>
<td valign="top">

FDE Domain PMs, FDE Application Engineers, CIS T&O leads, customer Adoption Leads

</td>
</tr>
</table>

**Starting points by role:**

-   *FDE Domain PM or process owner*: [Agent Quality Framework](agent-quality-framework-5f15077.md), [Precision Guardrails](precision-guardrails-ae75c89.md), [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md), and [Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md) — quality framework, precision guardrails, CoE operations model, and customer enablement shape both the TO-BE process design and the post-go-live organizational commitments
-   *FDE Dev Architect or Application Engineer*: [Supervisor Agent Pattern](supervisor-agent-pattern-67417bb.md), [Security and Data Protection](security-and-data-protection-a403094.md), [Joule Studio Build Map](joule-studio-build-map-67ad030.md), [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md), [Production Operations](production-operations-e24808a.md), and [Agent Evaluation](agent-evaluation-bb08bc0.md)
-   *Customer Enterprise Architect*: [Agent Quality Framework](agent-quality-framework-5f15077.md), [Security and Data Protection](security-and-data-protection-a403094.md), [Precision Guardrails](precision-guardrails-ae75c89.md), and [Multi-Agent Orchestration](multi-agent-orchestration-a7957a5.md) — quality framework, security, guardrails, and multi-agent topology are the conversation anchors for governance discussions
-   *CIS T&O or CoE lead*: [CIS CoE Operations Model](cis-coe-operations-model-cba3b4d.md), [Production Operations](production-operations-e24808a.md), [Token Efficiency](token-efficiency-8449831.md), [Agent Evaluation](agent-evaluation-bb08bc0.md), and [Customer Enablement and Adoption](customer-enablement-adoption-318c7e2.md) — the operations model, production operations, token efficiency, online evaluation monitoring, and customer enablement define the CIS post-delivery responsibilities and cost management approach
-   *CIS colleague new to this concept*: read in order; [Agent Quality Framework](agent-quality-framework-5f15077.md) establishes the vocabulary the rest of the document builds on


<a name="loiocc824ede03f04c40a2de1624834cb769__section_key_constraints"/>

## Key Constraints and Design Principles

All designs in this concept operate within three non-negotiable constraints.

**Delivery runs on two orthogonal axes — never a single bundled path.**
*Axis 1 — orchestrator / Joule activation.* SAP-Managed Joule (SMJ, "THE BOX") is the default and the only activation path that receives new capabilities (Joule Work, Joule Studio 2.0, new LoB onboarding) for all new FDE engagements. Customer-Managed Joule (CMJ) is deprecated — a migration target, not a delivery choice — and now receives only AutoSQL, Knowledge Graph, and DPQuery. *Axis 2 — custom-agent runtime.* A custom agent runs either as BYOA (the customer's own BTP subaccount — LangGraph/AG2/CAP/CrewAI) or low-code in Joule Studio 2.0; both register to SMJ via A2A. BYOA is not a fallback: it is the standard runtime for productization-intent FDE agents. The two FDE delivery types map cleanly onto Axis 2: IAI CAE (customer-specific first, productize later) → a custom agent on the customer subaccount orchestrated by SMJ; Standard Industries Development (build into SAP standard) → Joule Studio 2.0 on SMJ. All agent designs in this document can be implemented on either Axis-2 runtime.

**Strong runtime governance is required — the mechanism depends on the Axis-2 runtime.**
For agents built low-code in Joule Studio 2.0 on SMJ, SAP's BAIP managed agent runtime provides governance automatically: security, observability, tenant isolation, and OpenTelemetry-based telemetry are built into the harness. For BYOA agents on the customer's BTP subaccount, the same governance goals apply but are achieved through team-configured BTP infrastructure (Cloud Foundry or Kyma), with the FDE team responsible for wiring auth, observability, and security controls; the agent is surfaced in SMJ through outbound A2A registration.

> ### Note:  
> **GA is phased, not binary**: the SMJ orchestrator is GA (since 2026-06-01), the Joule Studio 2.0 build environment is in phased GA, the managed custom-agent runtime is EAC-gated (free through Dec 2026, not broadly GA), and bidirectional A2A is transitional/not GA. Architect the BYOA + outbound-A2A path (functional today) and treat the full Joule Studio 2.0 managed runtime plus bidirectional A2A as an H1 2027 target. In both runtimes, BAIP is structured around three internal layers: a **Context layer** (the SAP Knowledge Graph with 7M+ data fields, unified across SAP and non-SAP systems), a **Build layer** (Joule Studio), and a **Governance layer** (SAP AI Agent Hub). The designs in this document operate across all three.

**Human-in-the-loop is the default.** Agents automate repeatable tasks; humans intervene at judgment points. `sapTaskCenter` is the implementation mechanism. Every design in this concept includes an explicit escalation path to a human — no agent operates with unilateral authority over high-stakes decisions.

**Two customer prerequisites must be verified in Discovery before agent delivery is feasible.** First, the customer must be on a clean-core cloud deployment (RISE with SAP or GROW with SAP). Second, the AI Core Extended service plan must be provisionable in the customer's BAIP region (required for generative AI hub access). If either condition is not met, an agent-based Industry AI engagement may be premature.

**Discovery runs as five sub-phases** — 1 Onboard, 2 Generate, 3 Define, 4 Prioritize, 5 Pre-Execute. This legend applies wherever later topics refer to Discovery sub-phases by number.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_discovery_checklist"/>

## Customer Prerequisites Discovery Checklist

Confirm all of the following with the customer IT admin and the FDE AI Architect during the Discovery phase. Unresolved items are blocking risks for the Development sprint.

**Table: Customer Prerequisites Discovery Checklist**

<table>
<tr>
<th valign="top">

Check

</th>
<th valign="top">

Question to Ask

</th>
<th valign="top">

Blocking If

</th>
</tr>
<tr>
<td valign="top">

Cloud deployment

</td>
<td valign="top">

Is the SAP landscape on RISE with SAP or GROW with SAP?

</td>
<td valign="top">

On-premise or Private Cloud with no RISE commitment → agent delivery is premature

</td>
</tr>
<tr>
<td valign="top">

Clean core

</td>
<td valign="top">

Are there custom ABAP modifications in the systems the agent will access?

</td>
<td valign="top">

Significant custom ABAP → integration complexity may exceed sprint scope

</td>
</tr>
<tr>
<td valign="top">

Joule access

</td>
<td valign="top">

Is Joule provisioned and accessible to end users in the target system?

</td>
<td valign="top">

Joule not provisioned → Joule Work integration cannot be validated in the sprint

</td>
</tr>
<tr>
<td valign="top">

AI Core Extended plan

</td>
<td valign="top">

Can AI Core Extended be provisioned in the customer's BTP subaccount?

</td>
<td valign="top">

Not available in the customer's region → GenAI hub access blocked; sovereign model gateway required

</td>
</tr>
<tr>
<td valign="top">

BAIP region coverage

</td>
<td valign="top">

Does BAIP / SMJ cover the customer's BTP region?

</td>
<td valign="top">

SMJ not available in-region → customer-managed (CMJ) orchestrator + BYOA runtime required

</td>
</tr>
<tr>
<td valign="top">

S/4HANA API exposure

</td>
<td valign="top">

Are the relevant SAP APIs (e.g., S/4HANA OData services) accessible from BTP with proper OAuth 2.0 configuration?

</td>
<td valign="top">

APIs not exposed → agent tool calls cannot be tested; plan for BTP connectivity setup in Week 0

</td>
</tr>
<tr>
<td valign="top">

Customer IT admin commitment

</td>
<td valign="top">

Is a dedicated customer IT admin available for the sprint?

</td>
<td valign="top">

No IT admin → BAIP registration, Joule configuration, and threshold calibration cannot proceed

</td>
</tr>
<tr>
<td valign="top">

Data access

</td>
<td valign="top">

What test data is available? Is CDH (Customer Data Hub) needed?

</td>
<td valign="top">

No test data access before Week 3 → eval suite start is delayed; confirm CDH access if real data is required

</td>
</tr>
<tr>
<td valign="top">

Regulatory constraints

</td>
<td valign="top">

Is this a sovereign or regulated deployment (CN40, NS2, KSA)?

</td>
<td valign="top">

Sovereign deployment → confirm BAIP/Joule availability before sprint commit

</td>
</tr>
</table>

If three or more checks are unresolved at the end of Discovery, escalate to the Delivery Lead before committing to a Development sprint scope — unresolved prerequisites become sprint blockers, not sprint tasks.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_ai_golden_path"/>

## The AI Golden Path

**The AI Golden Path defines mandatory requirements for all production agents.** SAP's AI Golden Path (v1.2, July 2026) is the authoritative guide for AI development across SAP, CIS, and FDE contexts. It specifies 14 mandatory Technical Requirements (TR1–TR14) that every production agent must satisfy before commercial release — see [Production Operations](production-operations-e24808a.md) for the full checklist. It also mandates specific platforms (BAIP, AppFND, Agent Gateway, Task Center for HITL).

The Golden Path defines an **approach-first** principle: before designing a custom agent, confirm that none of the following simpler solutions can address the use case:

1.  Rule-based/deterministic code or a SAP service
2.  Tabular ML via RPT-1
3.  GenAI Hub orchestration without autonomous tool use
4.  A Joule skill in the Agent Harness

A custom code-based agent is the right answer at step 5 only. FDE sprint scoping should challenge the agent assumption explicitly at the Discovery phase.

The Golden Path also mandates a CTO-level directive: classification and regression tasks on tabular data must start with SAP's RPT-1 model before any LLM-based approach is considered.

**RPT-1 approach-first assessment — Discovery questions**: before committing to an LLM/agent approach, use these questions during Discovery to determine whether RPT-1 (SAP's structured tabular ML model) is the right first step:

**Table: RPT-1 vs. LLM/Agent Decision Guide**

<table>
<tr>
<th valign="top">

Question

</th>
<th valign="top">

RPT-1 Likely Right If…

</th>
<th valign="top">

LLM/Agent Likely Right If…

</th>
</tr>
<tr>
<td valign="top">

Is the core task classification or regression on structured tabular data?

</td>
<td valign="top">

Yes — the input is rows/columns from SAP entities (materials, orders, equipment)

</td>
<td valign="top">

No — the task requires understanding natural language, context, or unstructured content

</td>
</tr>
<tr>
<td valign="top">

Is there historical labeled data available or constructible?

</td>
<td valign="top">

Yes — at least hundreds of examples exist or can be synthesized from transaction history

</td>
<td valign="top">

No — labeled examples are scarce; the task is inherently context-dependent

</td>
</tr>
<tr>
<td valign="top">

Does the output map to one of: category, score, quantity, flag?

</td>
<td valign="top">

Yes — a single predicted value drives the downstream action

</td>
<td valign="top">

No — the output requires natural language explanation, multi-step reasoning, or coordinating multiple tool calls

</td>
</tr>
<tr>
<td valign="top">

Can accuracy be validated deterministically?

</td>
<td valign="top">

Yes — a confusion matrix or RMSE against held-out data is the right eval

</td>
<td valign="top">

No — correctness requires semantic judgment; LLM-as-judge or human eval is needed

</td>
</tr>
<tr>
<td valign="top">

Does the use case appear in an Industry AI domain?

</td>
<td valign="top">

Autonomous Asset Management (failure prediction), Autonomous Commodity Management (price forecasting), Autonomous Regulated Manufacturing (batch classification) — all have strong RPT-1 fit for the quantitative sub-tasks

</td>
<td valign="top">

Use cases centered on planning, explanation, or cross-system coordination require the agent layer

</td>
</tr>
</table>

**Decision rule**: if the answers point to RPT-1, use RPT-1 for the structured prediction sub-task and wrap it as a deterministic tool in the agent's tool set (the agent calls the RPT-1 model as one of its Layer 2 reference services — see [Precision Guardrails](precision-guardrails-ae75c89.md)). Do not replace RPT-1 with an LLM for tabular tasks; this is a hard Golden Path requirement, not a suggestion.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_lightspeed"/>

## The CIS Reference Implementation: Project LightSpeed

The CIS Solution Acceleration team has built **Project LightSpeed** (`context/fde/project-lightspeed/`) — a 13-stage Claude Code plugin framework that implements the principles in this concept document as a working delivery toolchain. Teams starting an FDE engagement can use it directly: it automates the journey from raw customer discovery artifacts to a deployed, observable, Joule-connected multi-agent application. Where this concept describes the 'what' and 'why', Project LightSpeed is the 'how'.

### Engineering Principles in Practice

The following five engineering principles, as implemented in Project LightSpeed, are concrete expressions of the concept's design philosophy:

**Table: Project LightSpeed Engineering Principles**

<table>
<tr>
<th valign="top">

Principle

</th>
<th valign="top">

Description

</th>
</tr>
<tr>
<td valign="top">

**Artifact-Driven**

</td>
<td valign="top">

Every stage produces a versioned, hashed artifact — knowledge never lives only in a conversation

</td>
</tr>
<tr>
<td valign="top">

**Knowledge-Grounded**

</td>
<td valign="top">

All facts are resolved against SAP's canonical entity catalog before being used in agent design

</td>
</tr>
<tr>
<td valign="top">

**Human-in-the-Loop**

</td>
<td valign="top">

Approval gates at topology decisions, spec changes, and deploy steps

</td>
</tr>
<tr>
<td valign="top">

**Composable**

</td>
<td valign="top">

Each stage produces independent artifacts; stages can be re-run without restarting

</td>
</tr>
<tr>
<td valign="top">

**Production-First**

</td>
<td valign="top">

Deploy guardrails, OTel observability, and A2A compliance from the first prototype

</td>
</tr>
</table>

**Artifact-Driven** and **Composable** realize the concept's emphasis on audit trails and structured delivery. **Knowledge-Grounded** operationalizes the BAIP context layer and SAP Knowledge Graph. **Human-in-the-Loop** is the mechanism, not just the policy. **Production-First** ensures that governance, observability, and compliance are not retrofitted after delivery.


<a name="loiocc824ede03f04c40a2de1624834cb769__section_open_questions"/>

## Open Questions and Critique

Design questions that require legal, product, or domain expert input before this concept can be used in a specific customer context are tracked in [Open Questions](open-questions-8bbe2ba.md). Questions Q12–Q15 were the platform verification items for the supervisor workflows; all four have since been verified (Q12/Q13 resolved, Q14/Q15 confirmed as current platform gaps with documented fallbacks).

A structured critical review of the concept — covering unverified platform assumptions, internal tensions, methodological gaps, and missing elements — is maintained separately. The critique's resolution status covers findings addressed in the topics, findings requiring external verification, and findings accepted as-is.

**Related Information**  


[Agent Quality Framework](agent-quality-framework-5f15077.md)

[Joule Studio Overview](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Overview of Joule Studio, SAP's low-code environment for building AI agent workflows.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub — plan, build, observe, and govern AI agents across their full lifecycle.") :arrow_upper_right:

[Generative AI Hub in SAP AI Core](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/7db524ee75e74bf8b50c167951fe34a5.html "Generative AI Hub provides access to foundation models and orchestration capabilities within SAP AI Core.") :arrow_upper_right:

[AI Governance in SAP LeanIX](https://help.sap.com/docs/LEANIX/72d375467c1e4dcb872dfa2998b6328d/275887e57a441014876a9243fb67f92b.html "AI Governance capabilities in SAP LeanIX for enterprise architecture management and AI application inventory.") :arrow_upper_right:
