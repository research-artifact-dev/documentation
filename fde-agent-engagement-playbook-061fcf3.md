<!-- loio061fcf3beb9a4d0090c924138c649d03 -->

# FDE Agent Engagement Playbook — SAP CIS Edition

This playbook describes the three parallel delivery tracks — Build, Operate, and Enable — that structure every FDE agent engagement in the SAP CIS model. It provides checklists, exit criteria, role tables, and operating procedures to ensure customer teams can run, improve, and extend their agents independently after handover.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_purpose_model"/>

## Purpose and Delivery Model

An FDE engagement is complete only when the customer can run, improve, and extend its agents without SAP involvement. This playbook achieves that standard by running three tracks in parallel from day one, each with its own milestones and exit criteria.

The scope of "run, improve and extend" differs by engagement type and must be agreed within the first two weeks. In FDE engagements, the agent becomes a standard SAP product — SAP retains the IP and the customer runs and configures their instance of it. Prompt-level and threshold adjustments are within the customer's scope; structural changes such as new tools, new workflows, or new integrations require re-engaging the FDE team via a Microdelivery or a new sprint. In CAIA engagements, the customer owns the artifact entirely and can extend it without restriction. Record which model applies in the engagement's statement of work.

**Delivery tracks overview**

<table>
<thead>
<tr>
<th valign="top">

Track

</th>
<th valign="top">

Goal

</th>
<th valign="top">

Owned By

</th>
<th valign="top">

Done When

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

**Build**

</td>
<td valign="top">

Deliver an agent that moves a named business KPI

</td>
<td valign="top">

FDE lead, customer product owner

</td>
<td valign="top">

Agent live in production, meeting its agreed quality bar

</td>
</tr>
<tr>
<td valign="top">

**Operate**

</td>
<td valign="top">

Make the agent observable, testable, safe, and changeable

</td>
<td valign="top">

FDE engineer, customer platform lead

</td>
<td valign="top">

Operational readiness checklist passed

</td>
</tr>
<tr>
<td valign="top">

**Enable**

</td>
<td valign="top">

Give the customer the skills and confidence to own it

</td>
<td valign="top">

FDE lead, customer sponsor

</td>
<td valign="top">

Customer team ships a change and handles an incident unaided

</td>
</tr>
</tbody>
</table>

In the SAP FDE model, the FDE lead role splits across the Domain PM (Build and Enable track ownership) and Dev Architect (Operate track ownership). The customer product owner corresponds to the Agent Owner defined in the Adoption Strategy; the customer platform lead corresponds to the Customer IT Lead. "Meeting its agreed quality bar" means the Aeval offline eval suite passes the agreed success rate threshold and all TR1–TR14 technical requirements are met. Both the quality bar and the success rate threshold must be set during Discovery, not at go-live.

Agents require more operational discipline than traditional software. They degrade quietly, shift behaviour when models, prompts, or upstream data change, and often depend on knowledge held only by the people who built them. Operability and enablement are therefore deliverables, not handoff tasks.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_ownership_operating_model"/>

## Ownership and Operating Model

Settle who owns the agent in the first two weeks, before build starts. The most common failure mode is an agent that works at go-live and then has no owner.

For FDE engagements, the statement of work is the Feedback Agreement (FBA) — the default legal instrument for every FDE engagement. The FBA does not transfer IP; SAP retains ownership as the agent is productized. The agent owner and operating team must be recorded in the Adoption Strategy (Sections 2 and 3), a mandatory FDE handover artifact, rather than only in the kickoff presentation.

Agree and record the following in the statement of work:

-   **Operating team:** customer platform team, business-unit team, or AI centre of excellence, with named people.
-   **Support model:** who is paged, who triages a bad output, who approves prompt, tool, or model changes.
-   **FDE role after launch:** none, time-boxed hypercare (typically 4 to 8 weeks), or an advisory retainer. In SAP FDE, the hypercare period follows RTC EAC — the first go-live with Early Adopter Care contacts. After hypercare exit, ongoing operational support transfers to CIS Technology & Operations (CIS T&O), not back to the FDE team.
-   **Embedded customer engineers:** at least one customer engineer on the build team from week one. Teams that co-build can operate; teams that receive a handoff rarely can.
-   **Business KPI and quality bar:** the outcome the agent must move, and the eval threshold it must hold.

**Engagement roles**

<table>
<thead>
<tr>
<th valign="top">

Role

</th>
<th valign="top">

Customer Side

</th>
<th valign="top">

FDE Side

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Executive sponsor

</td>
<td valign="top">

Owns the outcome, unblocks decisions

</td>
<td valign="top">

Engagement lead

</td>
</tr>
<tr>
<td valign="top">

Agent owner

</td>
<td valign="top">

Prioritises improvements, approves changes

</td>
<td valign="top">

FDE lead

</td>
</tr>
<tr>
<td valign="top">

Engineering

</td>
<td valign="top">

Builds, deploys, maintains

</td>
<td valign="top">

FDE engineers (pairing)

</td>
</tr>
<tr>
<td valign="top">

Operations / support

</td>
<td valign="top">

Monitors, triages, runs runbooks

</td>
<td valign="top">

FDE on hypercare

</td>
</tr>
<tr>
<td valign="top">

Risk, security, compliance

</td>
<td valign="top">

Signs off guardrails, data handling, audit

</td>
<td valign="top">

FDE engineer as point of contact

</td>
</tr>
</tbody>
</table>

These roles map to the nine governance roles formalized in the SAP Adoption Strategy (Adoption Strategy Skill v2.4): Executive Sponsor, Adoption Lead, Business Lead, Customer IT Lead, Account Executive, Enterprise Architect, Discovery/Execution Lead, Product/Solution Manager, and Test Lead. The Adoption Strategy requires all nine roles named with specific individuals — not job titles — before productization sign-off.

For FDE engagements, the scope of changes the agent owner can approve independently covers prompt-level changes, threshold adjustments, and configuration within the product's design parameters. Changes that introduce new tools, new integrations, or new agent capabilities require a Microdelivery or a new sprint. Make this boundary explicit to the agent owner during the Enable track to avoid expectation gaps at handover.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_operate_track"/>

## Operate Track

The Operate track makes the agent something a customer team can see, test, change, and contain. Each area below has a minimum standard every engagement must meet.

### Observability

-   **Full traces** of every run: inputs, each model call, tool calls and results, final output, latency, and token cost. The BAIP managed agent runtime provides built-in OpenTelemetry tracing for every agent run — no custom observability tooling is required. The CIS supervisor agent (Workflow B) adds an additional monitoring layer that aggregates telemetry, evaluates quality signals, and escalates anomalies to the CIS T&O team.
-   **Business metrics:** task completion rate, escalation-to-human rate, user corrections, negative feedback rate, and the target KPI.
-   **Cost dashboards** per agent, workflow, and tenant, with budget alerts. Cost dashboards and budget alerts are managed through SAP AI Agent Hub using the AI Units billing model. Agent Action Metering (TR13) must be live before production go-live for accurate cost attribution — it is a mandatory pre-production requirement, not a post-launch addition.
-   **Alerts** on error rates, tool failures, latency spikes, and drift in quality signals, routed to the customer on-call. Workflow B alerts follow the escalation tree defined in the CIS T&O runbook; thresholds are calibrated during the EAC period and documented in the operational runbook handed over at productization.

### Evaluation Harness

The eval harness is the most important asset left behind. Without it, the customer cannot change anything safely, and the agent freezes or decays.

-   A **golden dataset** of representative and edge-case inputs with expected outcomes or grading rubrics, drawn from real usage. In the SAP toolchain, the golden dataset is the input to Aeval — SAP's mandatory agent evaluation framework (TR9). Test cases follow Aeval's BDD-style given/when/then format and are defined in Week 0 of the sprint, before any workflow code is written.
-   **Automated evals** (deterministic checks plus LLM-as-judge where appropriate) run via Aeval CLI in GitHub Actions on every prompt, tool, or model change (TR9 mandatory). A reference CI pipeline template is available in the AI Golden Path. Aeval supports both deterministic checks and LLM-as-judge scoring.
-   **Online evaluation:** sampled production traffic scored automatically via the DwC pipeline, automatically deployed by the App Foundation platform (also TR9 mandatory). This is separate from and complementary to the supervisor agent's real-time quality monitoring.
-   **Incident feedback:** every production failure becomes a new eval case. Each confirmed quality incident generates a new Aeval test case, maintained by the CIS T&O team through the continuous improvement loop.

### Versioning and Deployment

-   Prompts, instructions, tool definitions, model versions, and config live in source control. For the **JS2.0/SMJ path (default, November 2026)**: the agent is built in Joule Studio 2.0; workflow JSON definitions and prompt files are kept in source control; deployment is via the Joule Studio managed runtime (no capability bundle required; Joule Work integration is automatic). For the **customer-managed (CMJ) orchestrator path (fallback)**: the agent lives in the cookbook repository with git tags per release; deployment is via `joule deploy` with a four-file capability bundle (manifest, workflow YAML, prompt files, tool definitions).

> ### Note:  
> Delivery involves two orthogonal axes: the *orchestrator* is **SMJ by default** — a customer-managed CMJ orchestrator is the deprecated option, used only for sovereign regions, non-standard Kyma configurations, or in-progress engagements; the *custom-agent runtime* is **BYOA** (pro-code on the customer's BTP subaccount, registering to SMJ via A2A) **or Joule Studio 2.0**. BYOA is a standard runtime, not a fallback. Document the reason for any CMJ-orchestrator use in the productization package.

-   Model versions are selected and pinned via the SAP generative AI hub, which abstracts provider specifics across frontier LLMs. The upgrade process: update the model service selection in Joule Studio or CMJ configuration, run the Aeval offline eval suite against the new model, compare success rates, then promote through the BTP space pipeline if results are acceptable.
-   CI/CD in the SAP context: CI is the Aeval offline eval running in GitHub Actions on every change; CD is managed runtime deployment via Joule Studio 2.0 (JS2.0/SMJ default) or `joule deploy` with the four-file capability bundle (customer-managed CMJ orchestrator fallback). Environments follow a three-space BTP promotion model: dev → test → prod. Rollback means redeploying a prior version via the managed runtime (JS2.0/SMJ) or reverting to the previous git-tagged capability bundle (CMJ-orchestrator path). Canary and shadow deployments are not natively supported by the current BAIP runtime; the BTP space promotion model with EAC as the controlled go-live is the functional equivalent.
-   Runtime, secrets, and connectors are defined as infrastructure as code, managed via Terraform and btp-pilot. Secrets go through BTP Secret Management; connectors are defined in the BTP destination service and subaccount configuration.

### Controls and Guardrails

-   **Least-privilege tool access:** scoped credentials, read-only by default, approval gates for irreversible actions such as payments, deletions, and external messages. In the SAP architecture, this is enforced by the Agent Gateway — the policy enforcement point. Each agent has an application registration with scoped authorizations; users grant agents a bounded subset of their own permissions, never more. Tool calls use token exchange rather than full credential delegation. Approval gates for irreversible actions are delivered as SAP Task Center approval tasks.
-   **Guardrails:** input and output filtering, PII handling, and limits on iterations, spend, and runtime per task. Input and output filtering is enforced at the Agent Gateway. PII handling is governed by the SPII contract (TR7, mandatory). Iteration limits are set in the Joule Studio workflow configuration. Spend limits are AI Units budget caps in AI Agent Hub (TR13). Note the 60-second hard timeout on Joule agent invocations — workflows that may run longer must implement resumable patterns.
-   **Human-in-the-loop** queues that are staffed, with clear escalation paths. HITL is non-negotiable in SAP Industry AI design. The delivery mechanism is SAP Task Center: the agent creates an approval task that lands in the process owner's inbox. An unstaffed HITL queue is a production blocker, not a deferred risk. HITL thresholds must be tested as part of the eval suite — verify the gate fires correctly below, at, and above the configured threshold.
-   **Audit logs** signed off by the customer's security and compliance teams before launch. OpenTelemetry traces from the BAIP harness serve as the audit trail; every span includes `correlation_id`, tool calls, inputs, outputs, and authorization context. The ORD endpoint (TR6, `sap:ai:v1` policy level) registers the agent in the ORD Global Registry, documenting its capabilities, data accesses, and authorization requirements. Security sign-off must explicitly cover the authorization matrix, data classification under the SPII contract, and the audit trail structure.

### Dependencies

Agents break when an upstream API, data source, or credential changes. Document every dependency, its owner, and the agent's behaviour when it is unavailable. Add scheduled synthetic checks that exercise each critical tool.

In the SAP context, critical dependencies include the generative AI hub (model service), BTP destination service (backend API connections), SAP Business Data Cloud (CDH, if used as a RAG grounding corpus), and any S/4HANA or cloud solution APIs accessed via tool calls. The ORD endpoint (TR6) is the formal mechanism for documenting the agent's API dependencies. Scheduled synthetic checks can be implemented as test workflows in Joule Studio that exercise each registered tool and alert on failure.

### Runbooks

Each runbook is tested by the customer team before exit.

**Runbook scenarios**

<table>
<thead>
<tr>
<th valign="top">

Scenario

</th>
<th valign="top">

Runbook Covers

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Bad or unexpected outputs

</td>
<td valign="top">

Finding the run in traces, isolating prompt, tool or data cause, adding an eval case

</td>
</tr>
<tr>
<td valign="top">

Tool or upstream outage

</td>
<td valign="top">

Fallback behaviour, kill switch, customer communication

</td>
</tr>
<tr>
<td valign="top">

Cost spike

</td>
<td valign="top">

Cost dashboards, loop and prompt-size checks, budget caps

</td>
</tr>
<tr>
<td valign="top">

Behaviour change request

</td>
<td valign="top">

Edit prompt or tool, run evals, deploy through CI, monitor

</td>
</tr>
<tr>
<td valign="top">

Model upgrade or deprecation

</td>
<td valign="top">

Eval comparison, staged rollout, rollback

</td>
</tr>
<tr>
<td valign="top">

Security or data incident

</td>
<td valign="top">

Disabling tools, preserving logs, escalation to security

</td>
</tr>
</tbody>
</table>

In the SAP context, each scenario has a platform-specific starting point. For bad outputs, find the run in AI Agent Hub using the `correlation_id` from the OTel trace. For cost spikes, check AI Units consumption in AI Agent Hub and inspect the `sap.ai.agent.trigger.type` OTel span attribute (TR13) for unexpected trigger patterns. For behaviour change requests, every prompt change must pass the Aeval offline CI gate before reaching production, then deploy using the JS2.0/SMJ managed runtime (default) or `joule deploy` / the CMJ cookbook pipeline (fallback). For model upgrades, update the model service selection in the generative AI hub configuration and re-run the Aeval eval suite before promoting to the production BTP space. For security incidents, disable the agent's application registration in IAS, preserve OTel traces in the BAIP platform, and escalate to SAP Security alongside the customer's own security team.

Pair runbooks with architecture documentation and a decision log explaining why key choices were made. In the SAP FDE context, the decision log is a mandatory productization handover artifact. It records key architectural choices — SMJ orchestrator (default) versus a customer-managed CMJ orchestrator (and the reason if CMJ was chosen), the custom-agent runtime (BYOA or Joule Studio 2.0), model selection rationale, HITL threshold calibration — so future sprint teams or Microdelivery teams can understand the design rationale.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_enable_track"/>

## Enable Track

The Enable track moves the customer from "FDEs built this for us" to a state where the team can run, improve, and build agents independently. It starts in week one, runs alongside delivery, and is reported in every steering meeting.

In SAP FDE engagements, independent operation means the customer team can configure and operate their SAP product instance and request new capabilities through the established Microdelivery or sprint engagement process — not that they own or modify the underlying SAP product code. For Industry AI customers, the goal is the operational capability to run and evolve their agent portfolio within the SAP platform and product parameters with decreasing FDE involvement over time.

### Capability Assessment and Plan

In the first two weeks, assess who will own, operate, and extend the agent, and their current skills with LLMs, evals, observability, and the customer's platform. Agree an enablement plan with named participants, milestones, and exit criteria, alongside the delivery plan.

In the SAP FDE model, this assessment maps to Adoption Strategy Sections 3 and 4: Section 3 names the nine governance roles with specific individuals (not job titles); Section 4 assesses readiness across five areas — Technical, Data, Process, User, and Governance. Draft Sections 1–2 in Discovery weeks 1–3 and Sections 3–4 in weeks 4–6. The Adoption Strategy is a living document updated throughout the engagement and finalized at productization — not a one-time Discovery output.

### Role-Based Training

**Training matrix by audience**

<table>
<thead>
<tr>
<th valign="top">

Audience

</th>
<th valign="top">

What They Need

</th>
<th valign="top">

Format

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Executive sponsors

</td>
<td valign="top">

What the agent does, its limits, risk posture, KPIs, reading the business dashboard

</td>
<td valign="top">

Short briefing, quarterly reviews

</td>
</tr>
<tr>
<td valign="top">

Agent owners / product leads

</td>
<td valign="top">

Prioritising improvements, reading quality metrics, approving changes, managing scope

</td>
<td valign="top">

Workshops, shadowing FDE decisions

</td>
</tr>
<tr>
<td valign="top">

Engineers / platform team

</td>
<td valign="top">

Architecture, prompt and tool changes, evals, CI/CD, model upgrades, debugging from traces

</td>
<td valign="top">

Pairing, hands-on labs on the real codebase

</td>
</tr>
<tr>
<td valign="top">

Operators / support

</td>
<td valign="top">

Triaging bad outputs, runbooks, escalation, kill switches

</td>
<td valign="top">

Runbook walkthroughs, incident drills

</td>
</tr>
<tr>
<td valign="top">

Human-in-the-loop reviewers

</td>
<td valign="top">

What to approve or reject, giving feedback that improves the agent

</td>
<td valign="top">

Short practical sessions with real examples

</td>
</tr>
<tr>
<td valign="top">

End users

</td>
<td valign="top">

What the agent does well and badly, using it effectively, flagging problems

</td>
<td valign="top">

Short guides, in-product tips, office hours

</td>
</tr>
<tr>
<td valign="top">

Risk, security, compliance

</td>
<td valign="top">

Guardrails, audit logs, data handling, change approval

</td>
<td valign="top">

Review sessions, sign-off workshops

</td>
</tr>
</tbody>
</table>

For the SAP platform, each audience needs additional content. Engineers and platform team members need hands-on sessions covering Joule Studio, Aeval CLI usage, the BTP space promotion procedure, and AI Agent Hub navigation. Operators need to learn Workflow B alert interpretation and how to identify a `correlation_id` for escalation. HITL reviewers need a practical session in SAP Task Center covering the approval task format, the context presented with each task, and how to give feedback that produces a new eval case. Risk and compliance reviewers need a walkthrough of the ORD endpoint, SPII contract scope, IAS authorization matrix, and how to retrieve a specific transaction record from the OTel trace store.

### Gradual Release of Responsibility

Ownership shifts in four stages. Progression is based on demonstrated capability, not elapsed time.

**Ownership transfer stages**

<table>
<thead>
<tr>
<th valign="top">

Stage

</th>
<th valign="top">

Who Leads

</th>
<th valign="top">

Evidence to Move On

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

1. FDE builds, customer observes

</td>
<td valign="top">

FDE

</td>
<td valign="top">

Customer engineers attend design sessions and code reviews

</td>
</tr>
<tr>
<td valign="top">

2. Pair building

</td>
<td valign="top">

Shared

</td>
<td valign="top">

Customer engineer co-authors a feature and its evals

</td>
</tr>
<tr>
<td valign="top">

3. Customer leads, FDE supports

</td>
<td valign="top">

Customer

</td>
<td valign="top">

Customer ships a change through evals and CI, and handles an incident with FDE review

</td>
</tr>
<tr>
<td valign="top">

4. Customer independent

</td>
<td valign="top">

Customer

</td>
<td valign="top">

Customer ships and operates with no FDE involvement for an agreed period

</td>
</tr>
</tbody>
</table>

In CIS terminology, Stage 1 is Demonstration, Stage 2 is Shadowing, Stage 3 is Reverse Shadowing, and Stage 4 is Independent Operation. The CIS minimum exit criterion for productization handover sign-off is Stage 3: the customer team must complete at least one full operational cycle without FDE intervention before handover is considered complete. Stage 4 is the target; if not reached before the FDE team exits, document this as a risk in Adoption Strategy Section 7 and agree a 30-day target date for reaching independent operation.

For the Stage 3 evidence, the customer IT admin must independently complete a prompt change, run the Aeval offline eval suite, pass the quality gate, and deploy via the JS2.0/SMJ managed runtime (default) or `joule deploy` / the CMJ cookbook pipeline (fallback) — without guidance from the FDE team.

### Learning by Doing

-   **Incident drills (game days):** simulate a degraded tool, a quality regression, or a cost spike, and have the customer team resolve it using the runbooks.
-   **Change exercises:** the customer team makes a real improvement end to end, including new eval cases.
-   **Model upgrade rehearsal:** swap the model in the BTP dev subaccount and run the full upgrade procedure. In practice: update the model service selection in the generative AI hub configuration, re-run the Aeval offline eval suite, review the success rate delta, and — if acceptable — promote through the test space to production. The customer IT admin must complete the first three steps independently as the rehearsal evidence.
-   **Trace review sessions:** walk through good and bad production runs together to build intuition. Use the AI Agent Hub trace view: walk through a successful run to show the full span tree, tool call sequence, and token cost; a HITL escalation to show what threshold fired and why; and a quality regression to show how the Aeval online eval flagged it and how a new test case was added through the incident feedback loop.

### Enablement Assets Left Behind

-   Architecture docs, decision log, and runbooks
-   A playbook for building the next agent: patterns, templates, and the eval approach
-   Recorded walkthroughs of key workflows and debugging sessions
-   End-user guides and FAQs
-   Onboarding material for new members of the operating team

In the SAP FDE productization package, these map to specific artifacts with named owners: the IT Admin operational runbook (Markdown, jointly maintained by Customer IT Admin and CIS T&O), the CIS T&O runbook covering the six core scenarios, the Aeval test case suite in GitHub, end-user quick reference cards, the seven-audience training materials, and the Adoption Strategy document (all seven sections, signed by the Executive Sponsor). All assets must reside in systems the customer organization owns and can maintain — not in FDE team repositories or personal drives.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_adoption_lasting_capability"/>

## Adoption and Lasting Capability

An agent that is healthy but unused has not succeeded. Operationalising the agent includes changing how people work and building capability beyond the first agent.

### Adoption and Change Management

-   **Communication:** tell affected users what is changing, why, and what stays the same.
-   **Business champions** in each affected team to model use and gather feedback. In the SAP FDE model, this role corresponds to the Adoption Lead in the Adoption Strategy governance table — a named individual accountable for driving adoption within the business unit.
-   **Feedback loops:** one-click flagging of problems, with visible follow-through so users keep reporting.
-   **Adoption metrics** tracked beside quality metrics: active users, usage per workflow, and the target KPI. These are surfaced in the Joule analytics dashboard and AI Agent Hub. The target KPI and its baseline are defined in Adoption Strategy Section 2 and tracked throughout the engagement.
-   **Process redesign:** update workflows, roles, and SLAs to reflect what the agent now does, rather than bolting it onto the old process.

> ### Tip:  
> A high human override rate in the first 30 days is not automatically a quality problem — it often points to incomplete process redesign or insufficient HITL reviewer training. Track the override rate weekly during hypercare to distinguish user resistance (a training and change management issue) from systematic agent errors (a quality issue that requires a prompt improvement cycle).

### Community of Practice

For customers planning more than one agent, help stand up an internal AI centre of excellence or community of practice:

-   Shared standards for evals, guardrails, observability, and change approval
-   A reusable platform and templates for new agents
-   A review process for new agent proposals
-   Regular show-and-tells so learning spreads across teams

For FDE and Industry AI customers, the reusable platform is SAP's standardized BAIP runtime and Joule Studio — not a customer-built stack. What a multi-agent Industry AI customer can productively build is: shared internal governance standards for operating SAP agents, an internal review and prioritization process for requesting new FDE or Industry AI Domain engagements, and documentation of their agent portfolio and operating lessons. For CAIA customers, a customer-owned AI CoE with reusable templates and a full review process is applicable as described above.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_engagement_timeline"/>

## Engagement Timeline

All three tracks run from the first week, and each phase ends at a gate with explicit criteria.

**Engagement roadmap — 3 tracks, 4 phases, 3 gates**

<table>
<thead>
<tr>
<th valign="top">

Track

</th>
<th valign="top">

Phase 1: Mobilise (First Two Weeks)

</th>
<th valign="top">

Phase 2: Co-build (Build Sprints)

</th>
<th valign="top">

Phase 3: Launch (Go-live, Hypercare)

</th>
<th valign="top">

Phase 4: Hand Over (Customer-led)

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

**Build**

</td>
<td valign="top">

Scope use case; agree KPI and quality bar

</td>
<td valign="top">

Build agent with embedded customer engineers

</td>
<td valign="top">

Go live; fix issues found in production

</td>
<td valign="top">

Customer extends and builds next use cases

</td>
</tr>
<tr>
<td valign="top">

**Operate**

</td>
<td valign="top">

Name owner and on-call; map dependencies

</td>
<td valign="top">

Tracing, evals, CI/CD, guardrails; write runbooks

</td>
<td valign="top">

Alerts to customer on-call; FDE hypercare support

</td>
<td valign="top">

Weekly, monthly, quarterly reviews; customer on-call

</td>
</tr>
<tr>
<td valign="top">

**Enable**

</td>
<td valign="top">

Assess skills; agree enablement plan

</td>
<td valign="top">

Pair building; role-based training; trace reviews

</td>
<td valign="top">

Customer leads changes; incident and upgrade drills

</td>
<td valign="top">

Customer independent; 30/60/90 day checks

</td>
</tr>
<tr>
<td valign="top">

**Gate**

</td>
<td valign="top">

 

</td>
<td valign="top">

**Plan gate** — ownership agreed

</td>
<td valign="top">

**Go-live gate** — operational readiness

</td>
<td valign="top">

**Exit gate** — enablement readiness

</td>
</tr>
</tbody>
</table>

The go-live gate checks the agent can be operated; the exit gate checks the customer can operate it. Both checklists appear in the following section. Phase lengths vary by engagement; set them in the delivery plan.

The four phases map to the SAP FDE process as follows: Mobilise corresponds to Discovery Phases 1–2 (Onboard and Generate, weeks 1–2); Co-build covers Discovery Phases 3–5 plus the Develop sprint (weeks 3–6 for Discovery, approximately 12 further weeks for the sprint); Launch is triggered by RTC EAC — the first go-live with Early Adopter Care contacts, which also marks the start of Productization; Hand over is the Productization phase, which runs in parallel with the later Develop sprint weeks and formalizes at EAC sign-off. The three gates map to: Plan gate = FDE Gate at the end of Discovery Phase 4–5 (around week 5–6); Go-live gate = RTC EAC trigger; Exit gate = Productization sign-off.

> ### Note:  
> The Deployment phase was removed from the SAP FDE process in September 2026 — Productization now starts at RTC EAC, not at sprint end.


<a name="loio061fcf3beb9a4d0090c924138c649d03__section_readiness_exit"/>

## Readiness and Exit

Go-live and engagement exit are gated on evidence, not dates. Copy this checklist into each engagement and tick items as they are met.

### Operational Readiness (Gate for Go-live)

-   [ ] Named agent owner and on-call rotation in place
-   [ ] Traces, dashboards, and alerts live and routed to the customer team
-   [ ] Eval suite running in CI, with an agreed quality bar met
-   [ ] Prompts, tools, config, and model versions in source control; rollback tested
-   [ ] Guardrails, tool permissions, and approval gates configured
-   [ ] Security, compliance, and audit logging signed off
-   [ ] Cost budgets and alerts set
-   [ ] Dependencies documented with synthetic checks
-   [ ] Runbooks written for all core scenarios

In the SAP context: the named agent owner is the individual from Adoption Strategy Section 3; the on-call list is the Customer IT Admin plus the CIS T&O contact, handed over at productization. Traces, dashboards, and alerts live means BAIP harness OTel traces are active, AI Agent Hub observability is configured, and Workflow B alerts are routed to the customer on-call. Eval suite in CI means Aeval offline eval is running in GitHub Actions and passing the agreed success rate threshold (TR9 mandatory). Source control with rollback tested means workflow definitions and prompt files (JS2.0/SMJ default) or the capability bundle (customer-managed CMJ orchestrator fallback) are in a GitHub repository with tagged releases, and the customer IT admin has independently completed at least one rollback exercise. Guardrails, tool permissions, and approval gates means the Agent Gateway is configured with the SPII contract (TR7 and TR8), Task Center approval tasks are live and tested, and HITL threshold test cases are passing. Security sign-off covers IAS auth (TR3), ORD endpoint (TR6), SPII contract (TR7), and OTel traces (TR12). Cost budgets and alerts means the AI Units budget cap is set in AI Agent Hub and Agent Action Metering is live (TR13). Runbooks written means both the CIS T&O runbook and the IT Admin operational runbook are in customer-owned systems, not FDE team drives.

### Enablement Readiness (Gate for Exit)

-   [ ] Customer team has shipped at least one change end to end without FDE help
-   [ ] Customer team has resolved at least one incident drill using the runbooks
-   [ ] Customer team has rehearsed a model upgrade in staging
-   [ ] Role-based training delivered to every audience in the enablement plan
-   [ ] Enablement assets handed over and stored in the customer's systems
-   [ ] Adoption and KPI metrics tracked and reviewed by the agent owner

In the SAP context: "shipped a change end to end" means the customer IT admin has independently completed a prompt change, run the Aeval offline eval suite, passed the quality gate, and deployed via the JS2.0/SMJ managed runtime (default) or `joule deploy` / the CMJ pipeline (fallback) without FDE guidance — this is the Stage 3 (Reverse Shadowing) evidence. The incident drill uses a Workflow B synthetic alert; the customer team triages to resolution without FDE intervention using the CIS T&O runbook. The model upgrade rehearsal takes place in the BTP dev subaccount. Role-based training delivered means all seven audiences in the training matrix have received training in the appropriate format and materials are stored in customer-owned systems. KPI metrics tracked means the Adoption Strategy Section 2 KPIs are active in the agent owner's review cadence with a baseline established during EAC for the 30/60/90-day comparison.

### Hypercare and Post-Engagement Support

-   **Hypercare:** a fixed period, typically 4 to 8 weeks, with FDE involvement declining week by week and clear exit criteria.
-   **Ongoing options:** periodic health checks, advisory hours, support for model upgrades and new use cases. Support for new use cases routes through Microdelivery for incremental additions or a new FDE sprint for scope expansions.
-   **Follow-up reviews** at 30, 60, and 90 days after exit, covering quality, cost, adoption, and the target KPI. The focus areas differ: 30 days — technical stability confirmation; 60 days — adoption trend review (override rate trend, active user count); 90 days — business value assessment against Adoption Strategy Section 2 KPIs and hypercare exit decision.

Post-hypercare operational support transfers to CIS T&O, not back to the FDE team. The CIS T&O operating rhythm after hypercare exit: weekly reviews covering Workflow B alert log, AI Agent Hub flagged runs, open Task Center escalations, and the Aeval online eval score trend; monthly reviews covering AI Units consumption versus budget, Aeval offline eval pass rate trend, and the Adoption Strategy Section 2 KPI delta against baseline; quarterly reviews covering generative AI hub model suitability, SAP product roadmap alignment with upcoming Industry AI Domain releases, and the Adoption Strategy Section 7 risk register.

### Ongoing Operating Cadence for the Customer

**Operating review cadence**

<table>
<thead>
<tr>
<th valign="top">

Cadence

</th>
<th valign="top">

Review

</th>
</tr>
</thead>
<tbody>
<tr>
<td valign="top">

Weekly

</td>
<td valign="top">

Quality metrics, flagged runs, open incidents

</td>
</tr>
<tr>
<td valign="top">

Monthly

</td>
<td valign="top">

Cost, eval results, backlog of improvements

</td>
</tr>
<tr>
<td valign="top">

Quarterly

</td>
<td valign="top">

Model and vendor options, scope, business outcome delivered

</td>
</tr>
</tbody>
</table>

**Related Information**  


[Joule Studio](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/36d3e974446844a1b0ffb7af8fa9446c.html "SAP's AI-first development environment for building and deploying AI agents, intelligent apps, and automations.") :arrow_upper_right:

[SAP AI Agent Hub](https://help.sap.com/docs/AI_AGENT_HUB/225000cc2ce44a228b9534a0f1098a9b/8ebd704d35a746c180af003db1f5acb0.html "Introduction to SAP AI Agent Hub — discover, govern, and monitor AI agents across your enterprise landscape.") :arrow_upper_right:

[Developing with Joule Studio CLI](https://help.sap.com/docs/JOULESTUDIO/7e496fe6f81f4975aec07d9c1cbd2aef/a7bbbde0cf3b48f4a82a4403577a6b2c.html "Build Joule Studio solutions from a local IDE using the jl CLI, with full control over code structure and CI/CD integration.") :arrow_upper_right:
