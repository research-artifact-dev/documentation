<!-- loio7abe271d0a6c4991b1cff9c606f16e1c -->

# Customer Enablement and Adoption

The Enable track is a mandatory, parallel workstream alongside Build and Operate throughout every FDE engagement. Delivering an agent and enabling adoption of that agent are two distinct deliverables: the first is a software artifact; the second is an organizational capability. Both must be complete before an FDE engagement is done.



<a name="loio7abe271d0a6c4991b1cff9c606f16e1c__section_three_tracks"/>

## Three-Track FDE Delivery Model

**Three Parallel Delivery Tracks**


<table>
<tr>
<th valign="top">

Track

</th>
<th valign="top">

Goal

</th>
<th valign="top">

Owner

</th>
<th valign="top">

Done When

</th>
</tr>
<tr>
<td valign="top">

Build

</td>
<td valign="top">

Deliver an agent that moves a named business KPI

</td>
<td valign="top">

FDE Application Engineers + Dev Architect

</td>
<td valign="top">

Agent live in production; aeval offline eval suite passes agreed success rate threshold; all TR1–TR14 requirements met

</td>
</tr>
<tr>
<td valign="top">

Operate

</td>
<td valign="top">

Make the agent observable, testable, safe, and changeable

</td>
<td valign="top">

FDE Dev Architect + CIS T&O

</td>
<td valign="top">

Operational readiness checklist passed; supervisor workflows deployed and calibrated; runbook handed over

</td>
</tr>
<tr>
<td valign="top">

Enable

</td>
<td valign="top">

Give the customer the skills and confidence to own the agent

</td>
<td valign="top">

FDE Domain PM + Adoption Lead

</td>
<td valign="top">

Customer team ships a change and handles an incident without FDE support

</td>
</tr>
</table>



<a name="loio7abe271d0a6c4991b1cff9c606f16e1c__section_adoption_strategy"/>

## Adoption Strategy — The Formal Engagement Deliverable

The Adoption Strategy is the primary Enable track artifact. It is not a communication plan or a training calendar. It is a structured accountability document that defines what adoption success looks like, who is accountable, and how the customer organization will sustain and evolve the agent after the FDE team departs. The Adoption Strategy must be produced throughout the engagement lifecycle, not at handover:

**Adoption Strategy Production Timeline**


<table>
<tr>
<th valign="top">

Phase

</th>
<th valign="top">

Sections Produced

</th>
</tr>
<tr>
<td valign="top">

Discovery \(weeks 1–3\)

</td>
<td valign="top">

Sections 1–2: Use Case Details and Expected Business Value \(quantified, measurable\)

</td>
</tr>
<tr>
<td valign="top">

Discovery \(weeks 4–6\)

</td>
<td valign="top">

Section 3: Governance and Accountability \(nine roles named with specific individuals\); Section 7: Top Adoption Risks \(first draft\)

</td>
</tr>
<tr>
<td valign="top">

Develop \(weeks 1–6\)

</td>
<td valign="top">

Sections 4–6: Readiness Baseline, Deployment Strategy, Milestone Table — refined as agent takes shape

</td>
</tr>
<tr>
<td valign="top">

EAC and Productization

</td>
<td valign="top">

All sections reviewed and updated from EAC learnings; Executive Sponsor signature obtained

</td>
</tr>
</table>

An Adoption Strategy that fails the productization quality gate is a productization blocker, not a deferred action.



<a name="loio7abe271d0a6c4991b1cff9c606f16e1c__section_gradual_release"/>

## Gradual Release of Responsibility \(Four-Stage Model\)

The Enable track transfers operational responsibility to the customer using a sequential four-stage model. Do not skip stages under time pressure:

**Four-Stage Responsibility Transfer Model**


<table>
<tr>
<th valign="top">

Stage

</th>
<th valign="top">

What Happens

</th>
<th valign="top">

Who Leads

</th>
<th valign="top">

Stage is Complete When

</th>
</tr>
<tr>
<td valign="top">

Stage 1 — Demonstration

</td>
<td valign="top">

FDE team demonstrates the agent in full operational mode: normal operation, HITL approval, anomaly alert, rollback

</td>
<td valign="top">

FDE team

</td>
<td valign="top">

Customer team can accurately describe agent behavior in their own words

</td>
</tr>
<tr>
<td valign="top">

Stage 2 — Shadowing

</td>
<td valign="top">

Customer IT admin and process owner shadow FDE team during live operations; all decisions made by FDE team

</td>
<td valign="top">

FDE team

</td>
<td valign="top">

Customer team can predict FDE team's decisions before they are made 80% of the time

</td>
</tr>
<tr>
<td valign="top">

Stage 3 — Reverse Shadowing

</td>
<td valign="top">

Customer team takes the wheel; FDE team observes silently and may coach only after a decision, not before

</td>
<td valign="top">

Customer team

</td>
<td valign="top">

Customer team completes one full operational cycle \(including at least one alert or approval event\) without FDE intervention

</td>
</tr>
<tr>
<td valign="top">

Stage 4 — Independent Operation

</td>
<td valign="top">

Customer team operates without FDE support; FDE team available as escalation contact only

</td>
<td valign="top">

Customer team

</td>
<td valign="top">

Customer team handles three consecutive incidents or threshold adjustment requests independently

</td>
</tr>
</table>

**Enable exit criterion for FDE handover: Stage 3 minimum; Stage 4 target.**



<a name="loio7abe271d0a6c4991b1cff9c606f16e1c__section_training"/>

## Role-Based Training Matrix

Training is not one-size-fits-all. The Enable track delivers seven distinct training audiences:

**Training Matrix by Audience**


<table>
<tr>
<th valign="top">

Audience

</th>
<th valign="top">

Depth

</th>
<th valign="top">

Format

</th>
<th valign="top">

Key Content

</th>
</tr>
<tr>
<td valign="top">

Executive Sponsor

</td>
<td valign="top">

Conceptual

</td>
<td valign="top">

30-min briefing

</td>
<td valign="top">

Business outcomes, value case progress, escalation path, 30/60/90-day success definition

</td>
</tr>
<tr>
<td valign="top">

Business Lead / Process Owner

</td>
<td valign="top">

Functional

</td>
<td valign="top">

2-hour workshop

</td>
<td valign="top">

HITL responsibilities, threshold rationale, process KPI dashboard, how to request a change

</td>
</tr>
<tr>
<td valign="top">

Customer IT Admin

</td>
<td valign="top">

Technical

</td>
<td valign="top">

Half-day hands-on

</td>
<td valign="top">

Agent architecture, BAIP registrations, [Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html) configuration, Workflow B alert interpretation, rollback procedure, SAP AI Agent Hub navigation

</td>
</tr>
<tr>
<td valign="top">

Joule End Users

</td>
<td valign="top">

User-facing

</td>
<td valign="top">

1-hour guided session + quick reference card

</td>
<td valign="top">

How to invoke the agent in Joule, what to do when an [approval task arrives in Task Center](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/8b0fa61be9d14d33a6b8ceec4c1c403c.html), how to flag a wrong answer

</td>
</tr>
<tr>
<td valign="top">

CIS T&O Operators

</td>
<td valign="top">

Operational

</td>
<td valign="top">

Runbook handover session

</td>
<td valign="top">

Alert escalation tree, in-place fix procedures, when to trigger a new sprint, Topology A customer data access with consent

</td>
</tr>
<tr>
<td valign="top">

Customer Security / Compliance

</td>
<td valign="top">

Governance

</td>
<td valign="top">

1-hour evidence walkthrough

</td>
<td valign="top">

Authorization matrix, audit trail structure, how to pull a specific transaction record, data protection boundaries

</td>
</tr>
<tr>
<td valign="top">

Customer IT Support / Help Desk

</td>
<td valign="top">

Triage

</td>
<td valign="top">

Quick reference card + 30-min briefing

</td>
<td valign="top">

First-line triage checklist: is this an agent issue or an SAP platform issue? Who to call? How to identify the `correlation_id` for escalation?

</td>
</tr>
</table>

> ### Note:  
> All training materials must be delivered as durable artifacts in a system the customer organization owns and can maintain — not in a sprint archive or in the FDE team's heads. Materials that exist only in a presentation deck are not handover artifacts.

**Related Information**  


[Task Management in Joule Work](https://help.sap.com/docs/JOULE_WORK/621104a90a01407fbb2d1613db39a54d/8b0fa61be9d14d33a6b8ceec4c1c403c.html)

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html)

[CIS CoE Operations and Reusable Components](cis-coe-operations-and-reusable-components-363e20e.md)

[Overview and Strategic Context](overview-and-strategic-context-507783e.md)
