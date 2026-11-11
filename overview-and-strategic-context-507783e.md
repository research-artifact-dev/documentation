<!-- loio507783ee368340b39a390c4eccf7e0bc -->

# Overview and Strategic Context

This document is a design concept created by the CIS \(Customer Industry Solutions\) Technology and Operations team for building, monitoring, and governing AI agents within SAP's Industry AI and FDE \(Forward Deployed Engineering\) context. It is a reference architecture and design pattern collection — not production code — intended for use in FDE engagements, customer governance conversations, and internal CIS enablement.



<a name="loio507783ee368340b39a390c4eccf7e0bc__section_why"/>

## Why Agent Governance Matters

SAP's Industry AI portfolio deploys autonomous agents in the highest-stakes industrial processes: batch release decisions in regulated manufacturing, commodity hedging, pricing at scale, and asset maintenance strategies. These are processes where failure has regulatory, financial, or operational consequences.

AI agents are qualitatively different from conventional software. A traditional application either works or it does not; the failure mode is visible. An agent reasons, selects tools, and acts on its own judgment. It can appear to work — completing each individual step correctly — while still producing wrong outputs, following unsafe paths, or drifting in quality over time in ways that are not immediately observable.

Governing agents well is not an optional add-on to Industry AI delivery. It is the condition under which enterprises can trust agents to act autonomously on high-stakes decisions, and the foundation on which value-based contracts \(10–30% of yearly customer benefit\) can be justified.



<a name="loio507783ee368340b39a390c4eccf7e0bc__section_strategic"/>

## SAP Strategic Alignment

This concept sits at the intersection of three SAP strategic pillars:

-   **Autonomous Enterprise** — SAP's vision for AI agents executing end-to-end business processes across applications, data, and organizations. The Autonomous Suite ships with 224 specialized agents and 51 Joule Assistants across five domains \(Finance, Spend, SCM, HCM, CX\). Industry AI adds vertical depth for the most complex, sector-specific challenges.

-   **AI-Native North Star Architecture** — four layers: User Experience \(Joule Work\), Process \(apps as capability providers\), Foundation \(Knowledge Graph, Business Data Cloud, model services\), and Platform \(BAIP managed agent runtime\).

-   **BAIP \(Business AI Platform\)** — three sub-layers: Context \(SAP Knowledge Graph with 7M+ fields\), Build \(Joule Studio\), and Governance \(SAP AI Agent Hub\).



<a name="loio507783ee368340b39a390c4eccf7e0bc__section_design_questions"/>

## Thirteen Core Design Questions

The concept answers the core design questions that arise in every high-stakes agent deployment:

**Design Questions and Chapters**


<table>
<tr>
<th valign="top">

\#

</th>
<th valign="top">

Design Question

</th>
<th valign="top">

Chapter

</th>
</tr>
<tr>
<td valign="top">

1

</td>
<td valign="top">

How do we ensure an agent works correctly — at build time and at runtime?

</td>
<td valign="top">

Agent Quality Framework

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

Agent Quality Framework, System Architecture

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

System Architecture and Build Paths

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

Security and Data Protection

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

Security and Data Protection

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

Precision Guardrails and Governance

</td>
</tr>
<tr>
<td valign="top">

7

</td>
<td valign="top">

How do we manage agent lifecycle after productization?

</td>
<td valign="top">

CIS CoE Operations and Reusable Components

</td>
</tr>
<tr>
<td valign="top">

8

</td>
<td valign="top">

When does an Industry AI use case require multiple specialist agents?

</td>
<td valign="top">

Multi-Agent Orchestration and Production Operations

</td>
</tr>
<tr>
<td valign="top">

9

</td>
<td valign="top">

How do we version, roll back, and migrate agents across environments?

</td>
<td valign="top">

Multi-Agent Orchestration and Production Operations

</td>
</tr>
<tr>
<td valign="top">

10

</td>
<td valign="top">

Which design components can be reused across engagements?

</td>
<td valign="top">

CIS CoE Operations and Reusable Components

</td>
</tr>
<tr>
<td valign="top">

11

</td>
<td valign="top">

How do we manage token efficiency across FDE build-time and Industry AI runtime?

</td>
<td valign="top">

Token Efficiency

</td>
</tr>
<tr>
<td valign="top">

12

</td>
<td valign="top">

How do we evaluate agent quality systematically across an FDE sprint?

</td>
<td valign="top">

Agent Quality Framework

</td>
</tr>
<tr>
<td valign="top">

13

</td>
<td valign="top">

How do we ensure customer organizations can adopt and sustain the agent after FDE departs?

</td>
<td valign="top">

Customer Enablement and Adoption

</td>
</tr>
</table>



<a name="loio507783ee368340b39a390c4eccf7e0bc__section_constraints"/>

## Key Design Constraints

All designs in this concept operate within three non-negotiable constraints:

-   **All development uses Joule Studio** as the primary and mandatory foundation. Two valid delivery paths exist: Joule Studio \(Paths 1–2, strategic destination\) and the Custom-Managed Joule \(CMJ\) BYOA path \(Path 3, valid primary field delivery path for pro-code builds\). See [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html) for the full product overview.

-   **Every agent runs on the BAIP managed agent runtime** — security, observability, tenant isolation, and OpenTelemetry telemetry are provided by the harness, not built custom. The AI Golden Path \(v1.2, July 2026\) defines 14 mandatory Technical Requirements \(TR1–TR14\) that every production agent must satisfy before commercial release.

-   **Human-in-the-loop is the default.** Agents automate repeatable tasks; humans intervene at judgment points via [SAP Human in the Loop](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/bc403ba6bc114733929e034d9bb3dad4.html). Every design in this concept includes an explicit escalation path to a human — no agent operates with unilateral authority over high-stakes decisions.

> ### Note:  
> Two customer prerequisites must be verified in Discovery before agent delivery is feasible. First, the customer must be on a clean-core cloud deployment \(RISE with SAP or GROW with SAP\). Second, the AI Core Extended service plan must be provisionable in the customer's BAIP region. If either condition is not met, an agent-based Industry AI engagement may be premature.



<a name="loio507783ee368340b39a390c4eccf7e0bc__section_fde"/>

## FDE Delivery Context

FDE \(Forward Deployed Engineering\) is an SAP-funded co-innovation model \(approximately 90-day sprints, 300 Person Days\) that always becomes standard SAP product. IP remains with SAP; the customer signs a Feedback Agreement \(FBA\).

The Industry AI E2E delivery process has five phases: Qualification → Discovery → Value Proof → Value Delivery and Expansion → Run as Standard. Each phase is separated by a domain-lead-approved gate \(Gates 1–5\).

**FDE Delivery Pod Roles**


<table>
<tr>
<th valign="top">

Role

</th>
<th valign="top">

Responsibility

</th>
</tr>
<tr>
<td valign="top">

FD Delivery Lead

</td>
<td valign="top">

Engagement ownership; Build and Enable track accountability

</td>
</tr>
<tr>
<td valign="top">

FD AI Architect

</td>
<td valign="top">

Architecture decisions; Operate track ownership

</td>
</tr>
<tr>
<td valign="top">

FD AI Engineer \(×2\)

</td>
<td valign="top">

Agent workflow implementation; evaluation harness; Joule Studio build

</td>
</tr>
<tr>
<td valign="top">

FD Data Scientist and Engineer

</td>
<td valign="top">

Data pipeline; SAP Knowledge Graph integration; test data strategy

</td>
</tr>
<tr>
<td valign="top">

FD DevOpsSec

</td>
<td valign="top">

BTP provisioning; CI/CD; security controls; environment migration

</td>
</tr>
</table>

> ### Tip:  
> The **approach-first** principle from the AI Golden Path requires confirming that no simpler solution addresses the use case before designing a custom agent: \(1\) rule-based code or SAP service, \(2\) tabular ML via RPT-1, \(3\) GenAI Hub orchestration without autonomous tool use, \(4\) a Joule skill in the Agent Harness. A custom code-based agent is correct only at step 5.

**Related Information**  


[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html)

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html)

[SAP AI Agent Hub — Step-by-Step Guide](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/5938fa55ea1241dc8d786764aa610f4a.html)

[Agent Quality Framework](agent-quality-framework-af194ba.md)

[System Architecture and Build Paths](system-architecture-and-build-paths-a6daf78.md)
