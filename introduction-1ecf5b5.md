<!-- loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59 -->

# Introduction

SAP's Industry AI portfolio brings autonomous agents to high-stakes industrial processes — batch release, commodity hedging, pricing, and asset maintenance. This topic explains why governing agents well is the foundation for enterprise trust and value-based delivery, and establishes the design principles that apply across all topics in this series.


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_opportunity_and_risk"/>

## The Opportunity and the Risk

SAP's Industry AI portfolio brings autonomous agents to the most complex, high-stakes processes in industrial enterprises: batch release decisions in regulated manufacturing, commodity hedging positions, pricing recommendations at scale, asset maintenance strategies. These are processes where getting it right generates significant business value — and where getting it wrong has regulatory, financial, or operational consequences.

This is precisely what makes AI agents different from conventional software in these contexts. A traditional application either works or it doesn't; the failure mode is visible. An AI agent reasons, selects tools, and acts on its own judgment. It can appear to work — completing each individual step correctly — while still producing wrong outputs, following unsafe paths, or drifting in quality over time in ways that are not immediately visible.

Governing agents well is not an optional add-on to Industry AI delivery. It is the condition under which enterprises can trust agents to act autonomously on high-stakes decisions, and the foundation on which value-based contracts (10–30% of yearly customer benefit) can be justified.


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_sap_strategic_platform_context"/>

## SAP Strategic Platform Context

The following diagram shows how end users, SAP Joule, Joule Studio, and the SAP Business AI Platform (BAIP) connect — from the user surface through the managed runtime and Knowledge Graph to the foundation models below.

```mermaid
graph TB
  User[End User] --> Joule[SAP Joule]
  Joule --> JouleStudio[Joule Studio]
  JouleStudio --> BAIP[SAP Business AI Platform - BAIP]
  BAIP --> Runtime[Managed Agent Runtime]
  BAIP --> AICore[SAP AI Core - Generative AI Hub]
  BAIP --> KnowledgeGraph[Knowledge Graph - 7M+ Business Fields]
  BAIP --> AgentHub[SAP AI Agent Hub]
  Runtime --> Supervisor[Supervisor Agent]
  Runtime --> Workers[Worker Agents]
  Supervisor --> TaskCenter[sapTaskCenter - Human Approval]
  Workers --> BTPServices[SAP BTP Services]
  AICore --> Models[Foundation Models]
```

This document was developed by CIS (Customer Industry Solutions) Technology and Operations team as a design concept for building, monitoring, and governing AI agents within SAP's Industry AI and FDE (Forward Deployed Engineering) context. SAP positions Industry AI as a fifth element of the Autonomous Enterprise stack — providing built-in vertical expertise with sector-specific process logic, data models, and regulatory logic on top of the generic agentic platform. CIS is the delivery engine for this layer: the central, customer-anchored Industry AI acceleration engine that instantiates these domain-specific capabilities with and for strategic customers.

This is not production code; it is a reference architecture and design pattern collection intended for use in:

-   **FDE engagements**: as a design reference during Discovery and Development phases
-   **Customer conversations**: as a framework for discussing agent governance with enterprise architects and process owners
-   **Internal enablement**: as a shared language for CIS practitioners designing Industry AI solutions

The concept answers the core design questions that arise in every high-stakes agent deployment:

**Caption: Design Questions and Topic References**

<table>
<tr>
<th valign="top">#</th>
<th valign="top">Design Question</th>
<th valign="top">Topic</th>
</tr>
<tr>
<td valign="top">

1

</td>
<td valign="top">

How do we ensure an agent works correctly — at build time and at runtime?

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-8204dab.md)

</td>
</tr>
<tr>
<td valign="top">

2

</td>
<td valign="top">

How do we handle failures autonomously: self-correct vs. escalate?

</td>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-8204dab.md), [Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md)

</td>
</tr>
<tr>
<td valign="top">

3

</td>
<td valign="top">

Does a supervisor agent pattern make sense within SAP's architecture?

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md)

</td>
</tr>
<tr>
<td valign="top">

4

</td>
<td valign="top">

How do we protect customer data while still enabling operational monitoring?

</td>
<td valign="top">

[Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md), [Security and Data Protection](security-and-data-protection-e8e4c85.md)

</td>
</tr>
<tr>
<td valign="top">

5

</td>
<td valign="top">

How do we address security at development time AND at runtime?

</td>
<td valign="top">

[Security and Data Protection](security-and-data-protection-e8e4c85.md)

</td>
</tr>
<tr>
<td valign="top">

6

</td>
<td valign="top">

How do we enforce precision guardrails for business-critical numerical outputs?

</td>
<td valign="top">

[Precision Guardrails](precision-guardrails-0beada2.md)

</td>
</tr>
<tr>
<td valign="top">

7

</td>
<td valign="top">

How do we manage agent lifecycle after productization: monitor, maintain, and continuously improve?

</td>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-b96eb95.md)

</td>
</tr>
<tr>
<td valign="top">

8

</td>
<td valign="top">

When does an Industry AI use case require multiple specialist agents, and how do we orchestrate and govern them safely?

</td>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a4d6df6.md)

</td>
</tr>
<tr>
<td valign="top">

9

</td>
<td valign="top">

How do we version, roll back, and migrate agents across environments without service disruption?

</td>
<td valign="top">

[Production Operations](production-operations-42850d6.md)

</td>
</tr>
<tr>
<td valign="top">

10

</td>
<td valign="top">

Which design components can be reused across engagements, and what constitutes a CIS Starter Kit for efficient delivery at scale?

</td>
<td valign="top">

[Reusable Components and Reference Implementations](reusable-components-1dca6eb.md)

</td>
</tr>
<tr>
<td valign="top">

11

</td>
<td valign="top">

How do we manage token efficiency across FDE build-time and Industry AI runtime to maintain a sustainable cost per correct outcome?

</td>
<td valign="top">

[Token Efficiency in Industry AI Delivery](token-efficiency-2d8ea45.md)

</td>
</tr>
<tr>
<td valign="top">

12

</td>
<td valign="top">

How do we evaluate agent quality systematically across an FDE sprint — from first prototype through to production monitoring?

</td>
<td valign="top">

[Agent Evaluation in FDE Sprints](agent-evaluation-f940c93.md)

</td>
</tr>
<tr>
<td valign="top">

13

</td>
<td valign="top">

How do we ensure customer organizations are actually able to adopt and sustain the agent after the FDE team departs?

</td>
<td valign="top">

[Customer Enablement and Adoption](customer-enablement-and-adoption-138ab79.md)

</td>
</tr>
</table>


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_reading_guide"/>

## Reading Guide

Each topic is self-contained but builds on the previous. Topics 1–4 cover design concepts; Topic 5 maps all concepts to concrete Joule Studio implementation choices; Topics 6–8 address the operational and organizational layers — how agents are governed, operated, orchestrated, and evolved after go-live.

**Caption: Topic Overview and Primary Audience**

<table>
<tr>
<th valign="top">Topic</th>
<th valign="top">What It Covers</th>
<th valign="top">Primary Audience</th>
</tr>
<tr>
<td valign="top">

[Agent Quality Framework](agent-quality-framework-8204dab.md)

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

[Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md)

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

[Security and Data Protection](security-and-data-protection-e8e4c85.md)

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

[Precision Guardrails](precision-guardrails-0beada2.md)

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

[Joule Studio Build Map](joule-studio-build-map-8dcb5ea.md)

</td>
<td valign="top">

Concrete implementation of all design concepts using Joule Studio low-code (n8n) nodes; master build map table; FDE build sequence

</td>
<td valign="top">

FDE application engineers, dev architects

</td>
</tr>
<tr>
<td valign="top">

[CIS CoE Operations Model](cis-coe-operations-model-b96eb95.md)

</td>
<td valign="top">

Lifecycle management after productization; supervisor as autonomous CIS proxy; operating rhythm, severity tiers, and continuous improvement loop; CoE maturity levels

</td>
<td valign="top">

FDE PMs, CIS T&amp;O leads, customer IT admins

</td>
</tr>
<tr>
<td valign="top">

[Multi-Agent Orchestration](multi-agent-orchestration-a4d6df6.md)

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

[Production Operations](production-operations-42850d6.md)

</td>
<td valign="top">

Agent versioning and rollback (five-component version stack); cost and token monitoring; AI Units billing model; environment migration (dev → test → prod); FDE delivery constraints

</td>
<td valign="top">

FDE dev architects, CIS T&amp;O leads

</td>
</tr>
<tr>
<td valign="top">

[Reusable Components and Reference Implementations](reusable-components-1dca6eb.md)

</td>
<td valign="top">

Three-layer reusability stack; component-by-component analysis; CIS Starter Kit definition; APM per-project setup; Discovery parameterization guide

</td>
<td valign="top">

FDE architects, CIS T&amp;O leads

</td>
</tr>
<tr>
<td valign="top">

[Token Efficiency in Industry AI Delivery](token-efficiency-2d8ea45.md)

</td>
<td valign="top">

Two cost pools (build-time FDE engineers vs. runtime deployed agents); SAP L0–L6 optimization framework; build-time engineer guidelines; runtime efficiency

</td>
<td valign="top">

FDE dev architects, FDE Application Engineers, CIS T&amp;O leads

</td>
</tr>
<tr>
<td valign="top">

[Agent Evaluation in FDE Sprints](agent-evaluation-f940c93.md)

</td>
<td valign="top">

Testing vs. evaluation distinction; SAP aeval framework; test case DSL as a sprint deliverable; Quality Flywheel mapped to sprint weeks 1–12; success rate as primary release gate

</td>
<td valign="top">

All — FDE Application Engineers and Dev Architects especially

</td>
</tr>
<tr>
<td valign="top">

[Customer Enablement and Adoption](customer-enablement-and-adoption-138ab79.md)

</td>
<td valign="top">

The Enable track as a mandatory parallel workstream; Adoption Strategy as formal handover artifact; four-stage gradual release of responsibility; role-based training for 7 audiences

</td>
<td valign="top">

FDE Domain PMs, FDE Application Engineers, CIS T&amp;O leads, customer Adoption Leads

</td>
</tr>
</table>

**Starting points by role:**

-   *FDE Domain PM or process owner*: [Agent Quality Framework](agent-quality-framework-8204dab.md), [Precision Guardrails](precision-guardrails-0beada2.md), [CIS CoE Operations Model](cis-coe-operations-model-b96eb95.md), and [Customer Enablement and Adoption](customer-enablement-and-adoption-138ab79.md) — quality framework, precision guardrails, CoE operations model, and customer enablement shape both the TO-BE process design and the post-go-live organizational commitments
-   *FDE Dev Architect or Application Engineer*: [Supervisor Agent Pattern](supervisor-agent-pattern-53ccc55.md), [Security and Data Protection](security-and-data-protection-e8e4c85.md), [Joule Studio Build Map](joule-studio-build-map-8dcb5ea.md), [Multi-Agent Orchestration](multi-agent-orchestration-a4d6df6.md), [Production Operations](production-operations-42850d6.md), and [Agent Evaluation in FDE Sprints](agent-evaluation-f940c93.md)
-   *Customer Enterprise Architect*: [Agent Quality Framework](agent-quality-framework-8204dab.md), [Security and Data Protection](security-and-data-protection-e8e4c85.md), [Precision Guardrails](precision-guardrails-0beada2.md), and [Multi-Agent Orchestration](multi-agent-orchestration-a4d6df6.md) — quality framework, security, guardrails, and multi-agent topology are the conversation anchors for governance discussions
-   *CIS T&O or CoE lead*: [CIS CoE Operations Model](cis-coe-operations-model-b96eb95.md), [Production Operations](production-operations-42850d6.md), [Token Efficiency in Industry AI Delivery](token-efficiency-2d8ea45.md), [Agent Evaluation in FDE Sprints](agent-evaluation-f940c93.md), and [Customer Enablement and Adoption](customer-enablement-and-adoption-138ab79.md) — the operations model, production operations, token efficiency, online evaluation monitoring, and customer enablement define the CIS post-delivery responsibilities and cost management approach


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_key_constraints"/>

## Key Constraints and Design Principles

All designs in this concept operate within three non-negotiable constraints.

**Delivery runs on two orthogonal axes — never a single bundled path.**

*Axis 1 — orchestrator / Joule activation.* SAP-Managed Joule (SMJ, "THE BOX") is the default and the only activation path that receives new capabilities (Joule Work, Joule Studio 2.0, new LoB onboarding) for all new FDE engagements. Customer-Managed Joule (CMJ) is deprecated — a migration target, not a delivery choice — and now receives only AutoSQL, Knowledge Graph, and DPQuery.

*Axis 2 — custom-agent runtime.* A custom agent runs either as BYOA (the customer's own BTP subaccount — LangGraph/AG2/CAP/CrewAI) or low-code in Joule Studio 2.0; both register to SMJ via A2A. BYOA is not a fallback: it is the standard runtime for productization-intent FDE agents. The two FDE delivery types map cleanly onto Axis 2: IAI CAE (customer-specific first, productize later) uses a custom agent on the customer subaccount orchestrated by SMJ; Standard Industries Development (build into SAP standard) uses Joule Studio 2.0 on SMJ. All agent designs in this document can be implemented on either Axis-2 runtime.

**Strong runtime governance is required — the mechanism depends on the Axis-2 runtime.**

For agents built low-code in Joule Studio 2.0 on SMJ, SAP's BAIP managed agent runtime provides governance automatically: security, observability, tenant isolation, and OpenTelemetry-based telemetry are built into the harness. For BYOA agents on the customer's BTP subaccount, the same governance goals apply but are achieved through team-configured BTP infrastructure (Cloud Foundry or Kyma), with the FDE team responsible for wiring auth, observability, and security controls; the agent is surfaced in SMJ through outbound A2A registration.

GA is phased, not binary: the SMJ orchestrator is GA (since 2026-06-01), the Joule Studio 2.0 build environment is in phased GA, the managed custom-agent runtime is EAC-gated (free through Dec 2026, not broadly GA), and bidirectional A2A is transitional/not GA. Guidance for engagements starting now: architect the BYOA + outbound-A2A path (functional today) and treat the full Joule Studio 2.0 managed runtime plus bidirectional A2A as an H1 2027 target. In both runtimes, BAIP is structured around three internal layers: a **Context layer** (the SAP Knowledge Graph with 7M+ data fields, unified across SAP and non-SAP systems), a **Build layer** (Joule Studio), and a **Governance layer** (SAP AI Agent Hub). The designs in this document operate across all three.
([SAP Sapphire 2026](https://news.sap.com/2026/05/sap-sapphire-sap-unveils-autonomous-enterprise/ "SAP Sapphire 2026 announcement: SAP unveils Autonomous Enterprise capabilities including Business AI Platform and Joule Studio.") :arrow_upper_right:)

**Human-in-the-loop is the default.**

Agents automate repeatable tasks; humans intervene at judgment points. `sapTaskCenter` is the implementation mechanism. Every design in this concept includes an explicit escalation path to a human — no agent operates with unilateral authority over high-stakes decisions.

**Two customer prerequisites must be verified in Discovery before agent delivery is feasible.**

First, the customer must be on a clean-core cloud deployment (RISE with SAP or GROW with SAP) — custom ABAP modifications and legacy integration patterns limit agent effectiveness, and Joule access is exclusive to cloud customers. On-premise access requires a demonstrated RISE migration commitment. Second, the AI Core Extended service plan must be provisionable in the customer's BAIP region (required for generative AI hub access). If either condition is not met, an agent-based Industry AI engagement may be premature.
([SAPInsider, May 2026](https://sapinsider.org/articles/decoding-the-autonomous-enterprise-how-sap-anthropic-and-joule-are-rewriting-enterprise-architecture/ "SAPInsider article: Decoding the Autonomous Enterprise — how SAP, Anthropic, and Joule are rewriting enterprise architecture.") :arrow_upper_right:)

**Discovery runs as five sub-phases** — 1 Onboard, 2 Generate, 3 Define, 4 Prioritize, 5 Pre-Execute. This legend applies wherever later topics refer to Discovery sub-phases by number.


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_discovery_checklist"/>

## Customer Prerequisites Discovery Checklist

Confirm all of the following with the customer IT admin and the FDE AI Architect during the Discovery phase. Unresolved items are blocking risks for the Development sprint.

**Caption: Customer Prerequisites Discovery Checklist**

<table>
<tr>
<th valign="top">Check</th>
<th valign="top">Question to Ask</th>
<th valign="top">Blocking If</th>
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

Does BAIP (Business AI Platform) / SMJ cover the customer's BTP region?

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

Sovereign deployment → confirm BAIP/Joule availability per [Security and Data Protection](security-and-data-protection-e8e4c85.md) before sprint commit

</td>
</tr>
</table>

> ### Caution:  
> If three or more checks are unresolved at the end of Discovery, escalate to the Delivery Lead before committing to a Development sprint scope — unresolved prerequisites become sprint blockers, not sprint tasks.


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_ai_golden_path"/>

## The AI Golden Path

SAP's AI Golden Path (v1.2, July 2026) is the authoritative guide for AI development across SAP, CIS, and FDE contexts. It specifies 14 mandatory Technical Requirements (TR1–TR14) that every production agent must satisfy before commercial release — see [Production Operations](production-operations-42850d6.md) for the full checklist. It also mandates specific platforms (BAIP, AppFND, Agent Gateway, Task Center for HITL) and includes a CTO-level directive: classification and regression tasks on tabular data must start with SAP's RPT-1 model before any LLM-based approach is considered.

The Golden Path defines an **approach-first** principle: before designing a custom agent, confirm that none of the following simpler solutions can address the use case:

1.  Rule-based/deterministic code or a SAP service
2.  Tabular ML via RPT-1
3.  GenAI Hub orchestration without autonomous tool use
4.  A Joule skill in the Agent Harness

A custom code-based agent is the right answer at step 5 only. FDE sprint scoping should challenge the agent assumption explicitly at the Discovery phase.

**RPT-1 approach-first assessment — Discovery questions:**

Before committing to an LLM/agent approach, ask these questions during Discovery to determine whether RPT-1 (SAP's structured tabular ML model) is the right first step.

**Caption: RPT-1 vs. LLM/Agent Decision Guide**

<table>
<tr>
<th valign="top">Question</th>
<th valign="top">RPT-1 Likely Right If…</th>
<th valign="top">LLM/Agent Likely Right If…</th>
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

Autonomous Asset Management (failure prediction), Autonomous Commodity Management (price forecasting), Autonomous Regulated Manufacturing (batch classification) — all have strong RPT-1 fit for quantitative sub-tasks

</td>
<td valign="top">

Use cases centered on planning, explanation, or cross-system coordination require the agent layer

</td>
</tr>
</table>

> ### Note:  
> If the answers point to RPT-1, use RPT-1 for the structured prediction sub-task and wrap it as a deterministic tool in the agent's tool set. Do not replace RPT-1 with an LLM for tabular tasks; this is a hard Golden Path requirement, not a suggestion.


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_project_lightspeed"/>

## The CIS Reference Implementation: Project LightSpeed

The CIS Solution Acceleration team has built **Project LightSpeed** — a 13-stage Claude Code plugin framework that implements the principles in this concept document as a working delivery toolchain. Teams starting an FDE engagement can use it directly: it automates the journey from raw customer discovery artifacts to a deployed, observable, Joule-connected multi-agent application. Where this concept describes the 'what' and 'why', Project LightSpeed is the 'how'. Cross-references to specific LightSpeed patterns are included throughout the relevant topics.

### Engineering Principles in Practice

The following five engineering principles, as implemented in Project LightSpeed, are concrete expressions of the concept's design philosophy.

**Caption: Project LightSpeed Engineering Principles**

<table>
<tr>
<th valign="top">Principle</th>
<th valign="top">Description</th>
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


<a name="loio1ecf5b5b0a5a4bd49ed60ed2d9c34d59__section_open_questions"/>

## Open Questions and Critique

Design questions that require legal, product, or domain expert input before this concept can be used in a specific customer context are tracked in [Open Questions](open-questions-ecf1afd.md). Questions Q12–Q15 were the platform verification items for the supervisor workflows; all four have since been verified (Q12/Q13 resolved, Q14/Q15 confirmed as current platform gaps with documented fallbacks).

A structured critical review of the concept — covering unverified platform assumptions, internal tensions, methodological gaps, and missing elements — is maintained separately. The critique's resolution status covers findings addressed in the topics, findings requiring external verification, and findings accepted as-is.

**Related Information**  


[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "SAP AI Agent Hub: central governance and lifecycle management platform for AI agents in the SAP Business AI Platform.") :arrow_upper_right:

[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "Joule Studio: low-code and pro-code environment for building, deploying, and governing SAP AI agents.") :arrow_upper_right:

[Generative AI Hub with SAP AI Core](https://help.sap.com/docs/AI_CORE/d5d03749687449ad9768e46ea9e32b1e/7db524ee75e74bf8b50c167951fe34a5.html "Generative AI Hub: access to foundation models through SAP AI Core, with data residency and governance controls.") :arrow_upper_right:

[Key Capabilities — SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/3f7ceafb1eb24d00ac50bf3499f50688.html "Key capabilities of SAP AI Agent Hub including agent observability, lifecycle management, and governance.") :arrow_upper_right:
