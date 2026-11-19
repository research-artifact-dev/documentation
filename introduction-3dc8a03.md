[//]: # (loio: 3dc8a03)

# Introduction: Why Agent Governance Matters for Industry AI

## The Opportunity and the Risk

SAP's Industry AI portfolio brings autonomous agents to the most complex, high-stakes
processes in industrial enterprises: batch release decisions in regulated manufacturing,
commodity hedging positions, pricing recommendations at scale, asset maintenance
strategies. These are processes where getting it right generates significant business
value — and where getting it wrong has regulatory, financial, or operational consequences.

This is precisely what makes AI agents different from conventional software in these
contexts. A traditional application either works or it doesn't; the failure mode is
visible. An AI agent reasons, selects tools, and acts on its own judgment. It can
appear to work — completing each individual step correctly — while still producing
wrong outputs, following unsafe paths, or drifting in quality over time in ways that
are not immediately visible.

Governing agents well is not an optional add-on to Industry AI delivery. It is the
condition under which enterprises can trust agents to act autonomously on high-stakes
decisions, and the foundation on which value-based contracts (10–30% of yearly
customer benefit) can be justified.

---

## What This Concept Addresses

This document was developed by CIS (Customer Industry Solutions) Technology and Operations team as a design concept
for building, monitoring, and governing AI agents within SAP's Industry AI and FDE
(Forward Deployed Engineering) context. SAP positions Industry AI as a fifth element
of the Autonomous Enterprise stack — providing "built-in vertical expertise with
sector-specific process logic, data models, and regulatory logic" on top of the generic
agentic platform. CIS is the delivery engine for this layer: the central,
customer-anchored Industry AI acceleration engine that instantiates these domain-specific
capabilities with and for strategic customers. It is not production code; it is a reference
architecture and design pattern collection intended for use in:

- **FDE engagements**: as a design reference during Discovery and Development phases
- **Customer conversations**: as a framework for discussing agent governance with
  enterprise architects and process owners
- **Internal enablement**: as a shared language for CIS practitioners designing
  Industry AI solutions

The concept answers the **core design questions** that arise in every high-stakes agent
deployment:

| # | Design question | Chapter | Key sections |
|---|---|---|---|
| 1 | How do we ensure an agent works correctly — at build time and at runtime? | [Ch 01 — Agent Quality Framework](agent-quality-framework-86a2b10.md) | §1.2 Three evaluation dimensions<br>§1.3 Build-time quality<br>§1.4 Runtime quality<br>§1.4.5 Iteration signals |
| 2 | How do we handle failures autonomously: self-correct vs. escalate? | [Ch 01](agent-quality-framework-86a2b10.md), [Ch 02](supervisor-agent-pattern-b40ee37.md) | Ch 01 §1.6 Escalation path<br>Ch 02 §2.3 Supervisor functions<br>Ch 02 §2.6 Escalation thresholds |
| 3 | Does a supervisor agent pattern make sense within SAP's architecture? | [Ch 02 — Supervisor Agent Pattern](supervisor-agent-pattern-b40ee37.md) | §2.1 Supervision tier<br>§2.2 Alternatives<br>§2.3 Three functions<br>§2.4 Architecture |
| 4 | How do we protect customer data while still enabling operational monitoring? | [Ch 02](supervisor-agent-pattern-b40ee37.md), [Ch 03](security-and-data-protection-e20205c.md) | Ch 02 §2.5 Data protection design<br>Ch 03 §3.8 Data protection architecture<br>Ch 03 §3.9 Sovereign AI<br>Ch 03 §3.11 Legal alignment |
| 5 | How do we address security at development time AND at runtime? | [Ch 03 — Security & Data Protection](security-and-data-protection-e20205c.md) | §3.3 Development-time security<br>§3.4 Runtime security<br>§3.5 Agent identity<br>§3.7 Prompt injection |
| 6 | How do we enforce precision guardrails for business-critical numerical outputs? | [Ch 04 — Precision Guardrails](precision-guardrails-3841e39.md) | §4.3 Guardrail stack<br>§4.4–§4.7 Four layers<br>§4.8 Confidence and uncertainty |
| 7 | How do we manage agent lifecycle after productization: monitor, maintain, and continuously improve? | [Ch 06 — CIS CoE Operations Model](cis-coe-operations-model-9406fa3.md) | §6.1 Lifecycle shift<br>§6.4 Supervisor as CIS proxy<br>§6.5 Operating rhythm<br>§6.6 Continuous improvement loop |
| 8 | When does an Industry AI use case require multiple specialist agents, and how do we orchestrate and govern them safely? | [Ch 07 — Multi-Agent Orchestration](multi-agent-orchestration-13725ca.md) | §7.1 When multi-agent<br>§7.2 SAP orchestration model<br>§7.3 Three topologies<br>§7.5 Quality extension<br>§7.7 Supervisor extension |
| 9 | How do we version, roll back, and migrate agents across environments without service disruption? | [Ch 08 — Production Operations](production-operations-324bb02.md) | §8.2 Versioning strategy<br>§8.3 Rollback mechanics<br>§8.6 Environment migration |
| 10 | Which design components can be reused across engagements, and what constitutes a CIS Starter Kit for efficient delivery at scale? | [Ch 09 — Reusable Components](reusable-components-560fb10.md) | §9.2 Three-layer reusability stack<br>§9.3 Component-by-component analysis<br>§9.4 CIS Starter Kit |
| 11 | How do we manage token efficiency across FDE build-time and Industry AI runtime to maintain a sustainable cost per correct outcome? | [Ch 10 — Token Efficiency](token-efficiency-f8f90a7.md) | §10.2 L0–L6 framework<br>§10.3 Build-time guidelines (FDE engineers)<br>§10.4 Runtime efficiency (deployed agents)<br>§10.5 Quality–efficiency connection |
| 12 | How do we evaluate agent quality systematically across an FDE sprint — from first prototype through to production monitoring? | [Ch 11 — Agent Evaluation in FDE Sprints](agent-evaluation-8cabb2d.md) | §11.3 Test case DSL<br>§11.5 Quality Flywheel (sprint weeks)<br>§11.7 Online evaluation in production<br>§11.8 Evaluation dimensions checklist |
| 13 | How do we ensure customer organizations are actually able to adopt and sustain the agent after the FDE team departs? | [Ch 12 — Customer Enablement & Adoption](customer-enablement-and-adoption-f7c118f.md) | §12.3 Adoption Strategy lifecycle<br>§12.4 Gradual release (4 stages)<br>§12.5 Role-based training (7 audiences)<br>§12.6 Enablement readiness checklist |

---

## How to Read This Document

Each chapter is self-contained but builds on the previous. Chapters 1–4 cover design
concepts; Chapter 5 maps all concepts to concrete Joule Studio implementation choices;
Chapters 6–8 address the operational and organizational layers — how agents are governed,
operated, orchestrated, and evolved after go-live.

| Chapter | What it covers | Primary audience |
|---|---|---|
| 1 — Agent Quality Framework | How to measure agent quality across three dimensions (technical, process KPI, business outcomes); build-time and runtime quality; the escalation path | All |
| 2 — Supervisor Agent Pattern | When and how to deploy a supervisor agent; three supervisor functions; worker-supervisor communication; data protection design | FDE architects, BAIP developers |
| 3 — Security & Data Protection | Agent-specific security threats; the three-tier defense model; agent identity and least-privilege; data flows and legal alignment | FDE architects, security reviewers |
| 4 — Precision Guardrails | The four-layer guardrail stack for numerical outputs; domain examples; human approval thresholds; audit trail requirements | FDE architects, process owners, compliance |
| 5 — Joule Studio Build Map | Concrete implementation of all design concepts using Joule Studio low-code (n8n) nodes; master build map table; FDE build sequence | FDE application engineers, dev architects |
| 6 — CIS CoE Operations Model | Lifecycle management after productization; supervisor as autonomous CIS proxy; operating rhythm, severity tiers, and continuous improvement loop; CoE maturity levels | FDE PMs, CIS T&O leads, customer IT admins |
| 7 — Multi-Agent Orchestration | When and how to compose multiple specialist agents; three orchestration topologies (flat, per-worker supervised, shared supervisor); supervisor patterns at scale | FDE architects, dev architects |
| 8 — Production Operations | Agent versioning and rollback (five-component version stack); cost and token monitoring; AI Units billing model; environment migration (dev → test → prod); FDE delivery constraints | FDE dev architects, CIS T&O leads |
| 9 — Reusable Components | Three-layer reusability stack (deploy as-is / parameterize / engagement-specific); component-by-component analysis; CIS Starter Kit definition; APM per-project setup; Discovery parameterization guide | FDE architects, CIS T&O leads |
| 10 — Token Efficiency | Two cost pools (build-time FDE engineers vs. runtime deployed agents); SAP L0–L6 optimization framework; build-time engineer guidelines (APM, caching, MCP gating, session handoff); runtime efficiency (Joule Studio design choices, supervisor as efficiency gate); quality–efficiency connection | FDE dev architects, FDE Application Engineers, CIS T&O leads |
| 11 — Agent Evaluation in FDE Sprints | Testing vs. evaluation distinction; SAP aeval framework (offline CI + online production monitoring, TR9 mandatory); test case DSL as a sprint deliverable; Quality Flywheel mapped to sprint weeks 1–12; success rate as primary release gate; evaluation dimensions checklist | All — FDE Application Engineers and Dev Architects especially |
| 12 — Customer Enablement & Adoption | The Enable track as a mandatory parallel workstream; Adoption Strategy as formal handover artifact (7 sections, 9 governance roles); four-stage gradual release of responsibility; role-based training for 7 audiences; enablement readiness checklist; CoE post-handover support model | FDE Domain PMs, FDE Application Engineers, CIS T&O leads, customer Adoption Leads |

**Starting points by role:**

- *FDE Domain PM or process owner*: Chapters 1, 4, 6, and 12 — quality framework, precision guardrails, CoE operations model, and customer enablement shape both the TO-BE process design and the post-go-live organizational commitments
- *FDE Dev Architect or Application Engineer*: Chapters 2, 3, 5, 7, 8, and 11 — supervisor pattern, security controls, Joule Studio build map, multi-agent orchestration, production operations, and the evaluation framework
- *Customer Enterprise Architect*: Chapters 1, 3, 4, and 7 — quality framework, security, guardrails, and multi-agent topology are the conversation anchors for governance discussions
- *CIS T&O or CoE lead*: Chapters 6, 8, 10, 11, and 12 — the operations model, production operations, token efficiency, online evaluation monitoring, and customer enablement define the CIS post-delivery responsibilities and cost management approach
- *CIS colleague new to this concept*: read in order; Chapter 1 establishes the vocabulary the rest of the document builds on

---

## Key Constraints and Design Principles

All designs in this concept operate within three non-negotiable constraints:

**Delivery runs on two orthogonal axes — never a single bundled "CMJ/BYOA" path.**
*Axis 1 — orchestrator / Joule activation.* SAP-Managed Joule (SMJ, "THE BOX") is the
default and the only activation path that receives new capabilities (Joule Work, Joule
Studio 2.0, new LoB onboarding) for all new FDE engagements. Customer-Managed Joule
(CMJ) is deprecated — a migration target, not a delivery choice — and now receives only
AutoSQL, Knowledge Graph, and DPQuery. *Axis 2 — custom-agent runtime.* A custom agent
runs either as BYOA (the customer's own BTP subaccount — LangGraph/AG2/CAP/CrewAI) or
low-code in Joule Studio 2.0; both register to SMJ via A2A. BYOA is not a fallback: it
is the standard runtime for productization-intent FDE agents. The two FDE delivery types
map cleanly onto Axis 2: IAI CAE (customer-specific first, productize later) → a custom
agent on the customer subaccount orchestrated by SMJ; Standard Industries Development
(build into SAP standard) → Joule Studio 2.0 on SMJ. All agent designs in this document
can be implemented on either Axis-2 runtime.

**Strong runtime governance is required — the mechanism depends on the Axis-2 runtime.**
For agents built low-code in Joule Studio 2.0 on SMJ, SAP's BAIP managed agent runtime
provides governance automatically: security, observability, tenant isolation, and
OpenTelemetry-based telemetry are built into the harness. For BYOA agents on the
customer's BTP subaccount, the same governance goals apply but are achieved through
team-configured BTP infrastructure (Cloud Foundry or Kyma), with the FDE team
responsible for wiring auth, observability, and security controls; the agent is surfaced
in SMJ through outbound A2A registration. **GA is phased, not binary**: the SMJ
orchestrator is GA (since 2026-06-01), the Joule Studio 2.0 build environment is in
phased GA, the managed custom-agent runtime is EAC-gated (free through Dec 2026, not
broadly GA), and bidirectional A2A is transitional/not GA. Guidance for engagements
starting now: architect the BYOA + outbound-A2A path (functional today) and treat the
full Joule Studio 2.0 managed runtime plus bidirectional A2A as an H1 2027 target (see
Chapter 6 section 6.8 for the full decision guide). In both runtimes, BAIP is structured
around three internal layers: a **Context layer** (the SAP Knowledge Graph with 7M+ data
fields, unified across SAP and non-SAP systems), a **Build layer** (Joule Studio), and
a **Governance layer** (SAP AI Agent Hub). The designs in this document operate across
all three.
([SAP Sapphire 2026](https://news.sap.com/2026/05/sap-sapphire-sap-unveils-autonomous-enterprise/))

**Human-in-the-loop is the default.** Agents automate repeatable tasks; humans
intervene at judgment points. `sapTaskCenter` is the implementation mechanism. Every
design in this concept includes an explicit escalation path to a human — no agent
operates with unilateral authority over high-stakes decisions.

**Two customer prerequisites must be verified in Discovery before agent delivery is
feasible.** First, the customer must be on a clean-core cloud deployment (RISE with SAP
or GROW with SAP) — custom ABAP modifications and legacy integration patterns limit
agent effectiveness, and Joule access is exclusive to cloud customers. On-premise access
requires a demonstrated RISE migration commitment. Second, the AI Core Extended service
plan must be provisionable in the customer's BAIP region (required for generative AI hub
access). If either condition is not met, an agent-based Industry AI engagement may be
premature.
([SAPInsider, May 2026](https://sapinsider.org/articles/decoding-the-autonomous-enterprise-how-sap-anthropic-and-joule-are-rewriting-enterprise-architecture/))

**Discovery runs as five sub-phases** — 1 Onboard, 2 Generate, 3 Define, 4 Prioritize,
5 Pre-Execute. This legend applies wherever later chapters refer to Discovery sub-phases
by number.

**Customer Prerequisites Discovery Checklist** — confirm all of the following with the
customer IT admin and the FDE AI Architect during the Discovery phase. Unresolved items
are blocking risks for the Development sprint.

| Check | Question to ask | Blocking if |
|---|---|---|
| Cloud deployment | Is the SAP landscape on RISE with SAP or GROW with SAP? | On-premise or Private Cloud with no RISE commitment → agent delivery is premature |
| Clean core | Are there custom ABAP modifications in the systems the agent will access? | Significant custom ABAP → integration complexity may exceed sprint scope |
| Joule access | Is Joule provisioned and accessible to end users in the target system? | Joule not provisioned → Joule Work integration cannot be validated in the sprint |
| AI Core Extended plan | Can AI Core Extended be provisioned in the customer's BTP subaccount? | Not available in the customer's region → GenAI hub access blocked; sovereign model gateway required (Ch5 §5.13) |
| BAIP region coverage | Does BAIP (Business AI Platform) / SMJ cover the customer's BTP region? | SMJ not available in-region → customer-managed (CMJ) orchestrator + BYOA runtime required; confirm per Ch6 §6.8 |
| S/4HANA API exposure | Are the relevant SAP APIs (e.g., S/4HANA OData services) accessible from BTP with proper OAuth 2.0 configuration? | APIs not exposed → agent tool calls cannot be tested; plan for BTP connectivity setup in Week 0 |
| Customer IT admin commitment | Is a dedicated customer IT admin available for the sprint? | No IT admin → BAIP registration, Joule configuration, and threshold calibration cannot proceed |
| Data access | What test data is available? Is CDH (Customer Data Hub) needed? | No test data access before Week 3 → eval suite start is delayed; confirm CDH access if real data is required |
| Regulatory constraints | Is this a sovereign or regulated deployment (CN40, NS2, KSA)? | Sovereign deployment → confirm BAIP/Joule availability per Ch3 §3.12 before sprint commit |

If three or more checks are unresolved at the end of Discovery, escalate to the Delivery Lead before committing to a Development sprint scope — unresolved prerequisites become sprint blockers, not sprint tasks.

**The AI Golden Path defines mandatory requirements for all production agents.** SAP's
AI Golden Path (v1.2, July 2026) is the authoritative guide for AI development across
SAP, CIS, and FDE contexts. It specifies 14 mandatory Technical Requirements (TR1–TR14)
that every production agent must satisfy before commercial release — see Chapter 8
section 8.10 for the full checklist. It also mandates specific platforms (BAIP, AppFND,
Agent Gateway, Task Center for HITL) and includes a CTO-level directive: classification
and regression tasks on tabular data must start with SAP's RPT-1 model before any
LLM-based approach is considered. The Golden Path further defines an **approach-first**
principle: before designing a custom agent, confirm that none of the following simpler
solutions can address the use case — (1) rule-based/deterministic code or a SAP service,
(2) tabular ML via RPT-1, (3) GenAI Hub orchestration without autonomous tool use,
(4) a Joule skill in the Agent Harness. A custom code-based agent is the right answer
at step 5 only. FDE sprint scoping should challenge the agent assumption explicitly at
the Discovery phase.

**RPT-1 approach-first assessment — Discovery questions**: before committing to an LLM/agent
approach, ask these questions during Discovery to determine whether RPT-1 (SAP's
structured tabular ML model) is the right first step:

| Question | RPT-1 likely right if... | LLM/agent likely right if... |
|---|---|---|
| Is the core task classification or regression on structured tabular data? | Yes — the input is rows/columns from SAP entities (materials, orders, equipment) | No — the task requires understanding natural language, context, or unstructured content |
| Is there historical labeled data available or constructible? | Yes — at least hundreds of examples exist or can be synthesized from transaction history | No — labeled examples are scarce; the task is inherently context-dependent |
| Does the output map to one of: category, score, quantity, flag? | Yes — a single predicted value drives the downstream action | No — the output requires natural language explanation, multi-step reasoning, or coordinating multiple tool calls |
| Can accuracy be validated deterministically? | Yes — a confusion matrix or RMSE against held-out data is the right eval | No — correctness requires semantic judgment; LLM-as-judge or human eval is needed |
| Does the use case appear in an Industry AI domain? | Autonomous Asset Management (failure prediction), Autonomous Commodity Management (price forecasting), Autonomous Regulated Manufacturing (batch classification) — all have strong RPT-1 fit for the quantitative sub-tasks | Use cases centered on planning, explanation, or cross-system coordination require the agent layer |

**Decision rule**: if the answers point to RPT-1, use RPT-1 for the structured prediction sub-task and wrap it as a deterministic tool in the agent's tool set (the agent calls the RPT-1 model as one of its Layer 2 reference services — see Chapter 4 §4.3). Do not replace RPT-1 with an LLM for tabular tasks; this is a hard Golden Path requirement, not a suggestion.

---

## The CIS Reference Implementation: Project LightSpeed

The CIS Solution Acceleration team has built **Project LightSpeed** (`context/fde/project-lightspeed/`) — a 13-stage Claude Code plugin framework that implements the principles in this concept document as a working delivery toolchain. Teams starting an FDE engagement can use it directly: it automates the journey from raw customer discovery artifacts to a deployed, observable, Joule-connected multi-agent application. Where this concept describes the 'what' and 'why', Project LightSpeed is the 'how'. Cross-references to specific LightSpeed patterns are included throughout the relevant chapters.

### Engineering Principles in Practice

The following five engineering principles, as implemented in Project LightSpeed, are concrete expressions of the concept's design philosophy. They represent the CIS delivery model putting these principles into practice:

| Principle | Description |
|---|---|
| **Artifact-Driven** | Every stage produces a versioned, hashed artifact — knowledge never lives only in a conversation |
| **Knowledge-Grounded** | All facts are resolved against SAP's canonical entity catalog before being used in agent design |
| **Human-in-the-Loop** | Approval gates at topology decisions, spec changes, and deploy steps |
| **Composable** | Each stage produces independent artifacts; stages can be re-run without restarting |
| **Production-First** | Deploy guardrails, OTel observability, and A2A compliance from the first prototype |

These principles complement the non-negotiable constraints established above. **Artifact-Driven** and **Composable** realize the concept's emphasis on audit trails and structured delivery. **Knowledge-Grounded** operationalizes the BAIP context layer and SAP Knowledge Graph. **Human-in-the-Loop** is the mechanism, not just the policy. **Production-First** ensures that governance, observability, and compliance are not retrofitted after delivery.

---

## Open Questions and Critique

Design questions that require legal, product, or domain expert input before this
concept can be used in a specific customer context are tracked in
[open-questions.md](open-questions-3a2265d.md). Questions Q12–Q15 were the platform verification
items for the supervisor workflows; all four have since been verified (Q12/Q13 resolved,
Q14/Q15 confirmed as current platform gaps with documented fallbacks).

A structured critical review of the concept — covering unverified platform assumptions,
internal tensions, methodological gaps, and missing elements — is in
concept-critique.md. The critique's resolution status is
maintained in that document: findings addressed in the chapters, findings requiring
external verification, and findings accepted as-is.
