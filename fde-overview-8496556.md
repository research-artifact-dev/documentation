<!-- loio8496556a55f84b25aec80131881b0d86 -->

# FDE Within SAP Industry AI: Overview

Orientation document for CIS team members preparing for FDE Alignment or onboarding discussions. Covers what FDE is, the Industry AI engagement process, the delivery pod structure, and where CIS fits within the broader model.

> ### Note:  
> All source material is internal — SAP and parties under NDA only.


<a name="loio8496556a55f84b25aec80131881b0d86__section_what_fde_is"/>

## What FDE Is

FDE (Forward Deployed Engineering) is an **SAP-funded ~90-day co-innovation sprint that always becomes standard SAP product.** Key characteristics:

-   **IP stays with SAP** — not a services engagement; the output is a product feature
-   **Customer is a pioneer** — they co-shape the product and commit to being a public reference
-   **SAP funds the squad** — customer signs an FBA (Feedback Agreement), not a paid services contract
-   Distinct from *Co-Innovation* (joint investment, shared ownership) and *CAIA / Custom AI Development* (customer-funded, stays customer-specific)

Three engagement sizes:

<br>

**FDE Engagement Sizes**

<table>
  <tr>
    <th valign="top">Size</th>
    <th valign="top">Duration</th>
    <th valign="top">Approval</th>
  </tr>
  <tr>
    <td valign="top">
Lean FDE
</td>
    <td valign="top">
&lt;90 days
</td>
    <td valign="top">
Standard
</td>
  </tr>
  <tr>
    <td valign="top">
Standard FDE
</td>
    <td valign="top">
~90 days
</td>
    <td valign="top">
Standard (majority of engagements)
</td>
  </tr>
  <tr>
    <td valign="top">
Large FDE
</td>
    <td valign="top">
&gt;90 days
</td>
    <td valign="top">
Board approval required
</td>
  </tr>
</table>

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_fde_process"/>

## The FDE Process

### Phases, Gates and Owners

Industry AI engagements follow the **Industry AI E2E process** — a lean, five-phase engagement methodology with a quality gate between each phase:

<br>

**Industry AI E2E Process Phases**

<table>
  <tr>
    <th valign="top">Phase</th>
    <th valign="top">Duration</th>
    <th valign="top">Lead Owner</th>
    <th valign="top">Key Activities</th>
  </tr>
  <tr>
    <td valign="top">
**Qualification**
</td>
    <td valign="top">
2–4 weeks
</td>
    <td valign="top">
Industry AI CRO
</td>
    <td valign="top">
Domain 3iaB nomination review; customer alignment & value case; qualification gate review & IAI Head/CRO approval
</td>
  </tr>
  <tr>
    <td valign="top">
**Discovery**
</td>
    <td valign="top">
2 weeks
</td>
    <td valign="top">
Industry Cluster
</td>
    <td valign="top">
Team staffing & discovery contracting; discovery workshop & engagement proposal
</td>
  </tr>
  <tr>
    <td valign="top">
**Value Proof**
</td>
    <td valign="top">
Up to 12 weeks
</td>
    <td valign="top">
Industry AI FDE
</td>
    <td valign="top">
Value-proof contracting; iterative MVP build (up to 11 weeks); value assessment; trafo engagement contracting
</td>
  </tr>
  <tr>
    <td valign="top">
**Value Delivery & Expansion**
</td>
    <td valign="top">
FDE delivery 8–10 weeks · FDE run continuous · productization varies
</td>
    <td valign="top">
Industry AI FDE (delivery); Industry Cluster (productization)
</td>
    <td valign="top">
Iterative build & deployment (7–9 weeks); value refinement; engagement expansion & upselling; commercialization under north-star setup
</td>
  </tr>
  <tr>
    <td valign="top">
**Run as Standard**
</td>
    <td valign="top">
Continuous
</td>
    <td valign="top">
Industry AI FDE
</td>
    <td valign="top">
Deployment of productized version; continuous run, maintenance & support
</td>
  </tr>
</table>

**Decision Gates** — each approved by the relevant Domain 3iaB lead (Customer fit → IBU domain lead; Scope & strategic fit → IC domain lead; Development prerequisites → IAI FDE team domain lead):

1.   **Gate 1** — Move into Qualification
2.   **Gate 2** — Move into Discovery
3.   **Gate 3** — Move into Value Proof
4.   **Gate 4** — Move into Value Delivery & Expansion
5.   **Gate 5** — Use case productization

MXP remains the system of record for tracking engagement state (including the "Handed over to CAIA" end state for use cases that move to custom delivery).

### MVP and Delivery Timing

-   **MVP** is built iteratively during **Value Proof** (up to ~11 weeks)
-   **Deployment & integration** happens during **Value Delivery & Expansion** (iterative build & deployment, ~7–9 weeks)

### Legal Agreement Progression

Each level unlocks the next phase:

1.   **FBA** (Feedback Agreement) — always required; enables first workshop
2.   **TEA / FTEA** — beta testing in SAP-managed environment
3.   **CA** (Collaboration Agreement) — customer's own environment or personal/productive data
4.   **Cloud Contract + SAP AI Terms** — production operation

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_fde_squad"/>

## FDE Squad — The Industry AI Delivery Pod

The **Industry AI FDE delivery pod** is a customer-facing, fast-moving, agile engineering team delivering high-value agentic, customer-specific solutions. Core pod roles (role names, not HR roles):

<br>

**Industry AI FDE Delivery Pod Roles**

<table>
  <tr>
    <th valign="top">Role</th>
    <th valign="top">Responsibility</th>
  </tr>
  <tr>
    <td valign="top">
**FD Delivery Lead**
</td>
    <td valign="top">
End-to-end engagement success: project setup, steering, governance, stakeholder alignment; owns value realization
</td>
  </tr>
  <tr>
    <td valign="top">
**FD Solution Strategist**
</td>
    <td valign="top">
Definition of agentic solutions anchored in customer business/operational context and success metrics
</td>
  </tr>
  <tr>
    <td valign="top">
**FD AI Architect**
</td>
    <td valign="top">
Prototype, value proof, and secure, scalable, enterprise-grade agentic solution architecture
</td>
  </tr>
  <tr>
    <td valign="top">
**FD AI Engineer** (typically ×2)
</td>
    <td valign="top">
Builds the agentic solution in the customer context; bridges process, application, AI, and deployment
</td>
  </tr>
  <tr>
    <td valign="top">
**FD Data Scientist & Engineer**
</td>
    <td valign="top">
Makes enterprise data usable, governed, observable, and reliable; data products, retrieval, pipelines, semantic/ontology
</td>
  </tr>
  <tr>
    <td valign="top">
**FD DevOpsSec**
</td>
    <td valign="top">
Operational backbone of agentic AI delivery; deploy, operate, and continuously improve at enterprise scale
</td>
  </tr>
</table>

The pod is supported by **business/advisory core roles** (virtual) — CRO team (commercial planning & GTM), Industry Cluster (E2E Product Manager and Product Manager for domain evolution & productization), and Value Advisor / Engagement Manager from the regions — and can be **augmented** by the Industry Cluster (Architect, Software/AI Engineer) and the CTO team (Foundation Data Scientist & Engineer, AI Reliability Engineer, Product Manager) for engagements with significant impact on the Industry AI stack. The number of delivery pods per engagement scales with requirements.

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_fde_industry_ai"/>

## FDE + Industry AI — The Specific Connection

**Industry AI is exclusively delivered via FDE.** There is no alternative delivery path for the 7 autonomous domains:

-   Autonomous Asset Management
-   Autonomous Commodity Management
-   Autonomous Adaptive Production
-   Autonomous Regulated Manufacturing
-   Autonomous Revenue Growth Management
-   Autonomous Unified Commerce
-   Autonomous Project Delivery

### What Makes Industry AI FDE Different

<br>

**General FDE vs. Industry AI FDE**

<table>
  <tr>
    <th valign="top">Aspect</th>
    <th valign="top">General FDE</th>
    <th valign="top">Industry AI FDE</th>
  </tr>
  <tr>
    <td valign="top">
Scope
</td>
    <td valign="top">
Any high-impact AI use case
</td>
    <td valign="top">
One of the 7 autonomous domains
</td>
  </tr>
  <tr>
    <td valign="top">
Commercial model
</td>
    <td valign="top">
SAP-funded sprint → product feature
</td>
    <td valign="top">
Value-based contract: flat annual subscription = 10–30% of yearly customer benefit
</td>
  </tr>
  <tr>
    <td valign="top">
Contract structure
</td>
    <td valign="top">
FBA + standard components
</td>
    <td valign="top">
Single bundled contract: Services + Agents & Assistants + Data & Ontologies + Business AI Platform
</td>
  </tr>
  <tr>
    <td valign="top">
Scale
</td>
    <td valign="top">
Varies
</td>
    <td valign="top">
Transformational — E2E autonomous business processes, cross-LoB
</td>
  </tr>
  <tr>
    <td valign="top">
Target customers
</td>
    <td valign="top">
Strategic, top-tier
</td>
    <td valign="top">
~10 enterprise customers in 2026 (pilot H1, commercial H2)
</td>
  </tr>
  <tr>
    <td valign="top">
Technology mandate
</td>
    <td valign="top">
SAP AI, Joule
</td>
    <td valign="top">
Industry AI Platform, Joule, agents productized into SAP portfolio
</td>
  </tr>
</table>

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_cis_role"/>

## Where CIS Fits

**CIS (Customer Industry Solutions) is the engineering delivery organization** within the FDE model. CIS provides the Application Engineers, Dev Architects, and Data Scientists who build the agents during FDE sprints.

CIS operates two tracks side by side:

<br>

**CIS Delivery Tracks**

<table>
  <tr>
    <th valign="top">Track</th>
    <th valign="top">Funding</th>
    <th valign="top">Output</th>
    <th valign="top">MXP End State</th>
  </tr>
  <tr>
    <td valign="top">
**FDE**
</td>
    <td valign="top">
SAP-funded
</td>
    <td valign="top">
Becomes standard SAP product
</td>
    <td valign="top">
Productization → product roadmap
</td>
  </tr>
  <tr>
    <td valign="top">
**CAIA** (Custom AI Application)
</td>
    <td valign="top">
Customer-funded
</td>
    <td valign="top">
Stays customer-specific
</td>
    <td valign="top">
"Handed over to CAIA"
</td>
  </tr>
</table>

CAIA is initiated when: (a) a customer completes an FDE Discovery but the use case does not qualify for full FDE productization, or (b) the customer wants to extend/customize a delivered FDE outcome for their own environment.

### CIS-Specific Standards and Tooling

CIS does not run a separate FDE process — it operates within SAP's standard FDE program. What is CIS-specific:

-   **Project LightSpeed** — CIS Solution Acceleration's delivery toolchain; 13-stage pipeline from discovery workshop to deployed multi-agent BTP application
-   **CIS CoE Operations Model** — ongoing support, monitoring, and improvement post-productization; see [CIS CoE Operations Model](cis-coe-operations-model-57b29ae.md)
-   **Agent Development Concept** — 12-chapter reference architecture covering quality, security, and governance standards applied to all agent builds; see [Introduction](introduction-c495d02.md) for the overview
-   **FDE Agent Engagement Playbook — SAP CIS Edition** — practitioner-facing operational guide for FDE delivery squads; translates concept design patterns into engagement checklists, timelines, and handover criteria; see [FDE Agent Engagement Playbook](fde-agent-engagement-playbook-afe3924.md)

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_fde_vs_caia"/>

## FDE vs. CAIA — Decision Guide

<br>

**FDE vs. CAIA Decision Criteria**

<table>
  <tr>
    <th valign="top">Question</th>
    <th valign="top">FDE</th>
    <th valign="top">CAIA</th>
  </tr>
  <tr>
    <td valign="top">
Who funds it?
</td>
    <td valign="top">
SAP
</td>
    <td valign="top">
Customer
</td>
  </tr>
  <tr>
    <td valign="top">
Does IP become SAP product?
</td>
    <td valign="top">
Yes
</td>
    <td valign="top">
No
</td>
  </tr>
  <tr>
    <td valign="top">
Is there a value-based contract?
</td>
    <td valign="top">
Yes (Industry AI)
</td>
    <td valign="top">
No
</td>
  </tr>
  <tr>
    <td valign="top">
Reusable across customers?
</td>
    <td valign="top">
Yes
</td>
    <td valign="top">
No
</td>
  </tr>
  <tr>
    <td valign="top">
What does the customer sign?
</td>
    <td valign="top">
FBA (+ TEA/CA as needed)
</td>
    <td valign="top">
Customer-funded services contract
</td>
  </tr>
  <tr>
    <td valign="top">
Triggered by...?
</td>
    <td valign="top">
CRO nomination + qualification
</td>
    <td valign="top">
FDE Discovery workshop finding, or direct customer request
</td>
  </tr>
</table>

---


<a name="loio8496556a55f84b25aec80131881b0d86__section_terminology"/>

## Key Terminology

<br>

**FDE Key Terms**

<table>
  <tr>
    <th valign="top">Term</th>
    <th valign="top">Definition</th>
  </tr>
  <tr>
    <td valign="top">
FDE
</td>
    <td valign="top">
Forward Deployed Engineering — SAP-funded sprint, IP stays with SAP
</td>
  </tr>
  <tr>
    <td valign="top">
CAIA
</td>
    <td valign="top">
Custom AI Application — customer-funded, customer-specific
</td>
  </tr>
  <tr>
    <td valign="top">
FBA
</td>
    <td valign="top">
Feedback Agreement — the customer contract for FDE
</td>
  </tr>
  <tr>
    <td valign="top">
MXP
</td>
    <td valign="top">
SAP's internal engagement tracking system
</td>
  </tr>
  <tr>
    <td valign="top">
RTC EAC
</td>
    <td valign="top">
Release-to-Customer / Early Adopter Care — first go-live with Early Adopter Care contacts; triggers Productization
</td>
  </tr>
  <tr>
    <td valign="top">
EAC
</td>
    <td valign="top">
Early Adopter Care
</td>
  </tr>
  <tr>
    <td valign="top">
IBU
</td>
    <td valign="top">
Industry Business Unit
</td>
  </tr>
  <tr>
    <td valign="top">
LoB
</td>
    <td valign="top">
Line of Business
</td>
  </tr>
  <tr>
    <td valign="top">
CRO Team
</td>
    <td valign="top">
Industry AI CRO — owns GTM, pipeline building, qualification and commercial engagement planning (Discovery is led by the Industry Cluster)
</td>
  </tr>
  <tr>
    <td valign="top">
LOB/AppAI
</td>
    <td valign="top">
Line of Business / Application AI — partners on productization and product-roadmap integration
</td>
  </tr>
</table>

**Related Information**  

[Introduction](introduction-c495d02.md)

[FDE Agent Engagement Playbook](fde-agent-engagement-playbook-afe3924.md)

[CIS CoE Operations Model](cis-coe-operations-model-57b29ae.md)

[Customer Enablement and Adoption](customer-enablement-and-adoption-ce2e74a.md)

[CIS Agent Development Overview](cis-agent-development-overview-5ca3b42.md)

[Joule Studio Documentation](https://help.sap.com/docs/joule-studio "Joule Studio documentation hub covering agent development, workflows, and enterprise application building.") :arrow_upper_right:

[SAP Generative AI Hub](https://help.sap.com/docs/sap-ai-core/generative-ai/generative-ai-hub "SAP Generative AI Hub: managed LLM integration within SAP AI Core for enterprise-grade model execution.") :arrow_upper_right:
