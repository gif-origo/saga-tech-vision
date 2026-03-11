# Saga Technical Vision 2026–2028

**Document Status:** Draft  
**Audience:** Development Team  
**Last Updated:** February 2026

---

## 1. Executive Summary

Saga is a multi-purpose healthcare platform serving the Icelandic health system, with ambitions to expand into the Nordic market. The system has evolved over many years, resulting in a layered architecture spanning legacy Delphi components, a WPF desktop client, and modern Angular-based modules (SagaPlus) backed by .NET services.

This document outlines a 1–2 year technical vision organized around **four product pillars** — Registration, Journal, Billing, and Scheduling — and supported by a **data sovereignty and transparency strategy** to address government concerns, and four **technical pillars**: modernizing the architecture (including a new multi-platform shell), adopting healthcare interoperability standards to meet EHDS regulatory requirements, improving developer experience, and hardening security. Together, these investments will position Saga for sustainable growth — both as a product and as a codebase the team can confidently evolve.

---

## 2. Where We Are Today

### 2.1 Architecture Landscape

Saga's current architecture reflects its history. The system consists of several layers that coexist in production:

- **Delphi desktop client** — the original UI layer and current application shell, still in active use for core workflows. Difficult to modify, test, and deploy. All other UI components (WPF, SagaPlus) are embedded within this shell.
- **WPF components** — a partial modernization of the desktop experience, sharing the client process with the Delphi layer.
- **SagaPlus (Angular)** — embedded web modules delivered inside the desktop shell, representing the newest user-facing functionality. Lives in a separate codebase.
- **Modern .NET backend services** — all backend development uses modern .NET, providing APIs for the SagaPlus modules and handling business logic, data access, and integrations.
- **Oracle Database** — the primary data store. Oracle is deeply embedded in the system and represents a significant operational cost. While there is no immediate migration plan, PostgreSQL is the preferred target if a database transition becomes a strategic priority.
- **RabbitMQ** — used for asynchronous messaging between services.

### 2.2 Deployment Landscape

Saga is currently deployed as multiple distributed instances, each serving different healthcare organizations. This multi-instance model has historically allowed organizations to operate independently, but it comes with significant costs: every deployment must be maintained, upgraded, and monitored separately, and configuration drift between instances creates testing and support complexity.

An ongoing strategic initiative is consolidating these separate deployments — joining organizations into fewer, shared instances. This consolidation represents one of the most impactful opportunities for the Saga team in the near term:

- **Reduced operational overhead.** Fewer instances means fewer deployments to manage, fewer environments to patch, and fewer configurations to keep in sync. This directly frees up developer and operations time for product work.
- **Faster, more confident releases.** With fewer deployment targets, the team can invest in a single, well-tested release pipeline rather than managing variations across many instances. Releases become more predictable and less risky.
- **Consistent feature rollout.** Consolidated instances make it feasible to roll out new SagaPlus modules (Sheets, Journal, Billing) to all organizations simultaneously rather than managing staggered rollouts across dozens of separate deployments.
- **Simplified testing.** Instead of testing against multiple instance configurations, the team can focus on a smaller number of representative environments, improving test coverage and reducing the cost of environment provisioning.
- **Foundation for multi-tenancy.** Consolidation naturally pushes the architecture toward multi-tenant patterns — shared infrastructure with organization-level data isolation, configurable workflows, and role-based access. Investing in proper multi-tenancy now will also support Nordic expansion, where new customers can be onboarded to an existing instance rather than requiring a dedicated deployment.

The consolidation strategy should be treated as an enabler for everything else in this vision. Architectural decisions — particularly around data isolation, configuration management, and deployment pipelines — should assume a consolidated deployment model as the target state. However, consolidation also concentrates risk — a single consolidated instance serving multiple organizations becomes a higher-value target for attackers and a larger blast radius for security incidents. This is addressed in Section 5.4.

### 2.3 Key Challenges

**Integration gaps and EHDS compliance.** Saga currently lacks standardized healthcare interoperability interfaces. Integrations with external systems (labs, imaging, national registries) are largely custom point-to-point connections. This creates maintenance burden and is a barrier to Nordic expansion. More urgently, the European Health Data Space (EHDS) regulation entered into force in March 2025 and will require EHR systems — including Saga — to support standardized data exchange formats by 2029. Our current interoperability gap is an emerging compliance risk.

**Testing and environment management.** The mixed technology stack makes it difficult to set up reliable test environments. The Delphi layer is particularly hard to test in isolation, and end-to-end testing across the full stack requires significant manual effort. Oracle licensing costs further complicate environment provisioning — spinning up multiple test or staging instances is expensive, limiting the team's ability to test in production-like conditions.

**Database cost and lock-in.** Oracle is a considerable operational expense and creates vendor lock-in. Oracle-specific features (PL/SQL procedures, proprietary syntax, specific data types) are likely embedded throughout the codebase, making a future migration non-trivial. While a full database migration is not the current priority, the cost and rigidity of Oracle is a strategic concern that should inform architectural decisions going forward.

**Desktop shell dependency.** All Saga UI components — including modern SagaPlus modules — are currently hosted inside the legacy Delphi application. This ties the entire user experience to a Windows-only, hard-to-maintain shell and blocks multi-platform expansion.

**User adoption of new modules.** SagaPlus modules represent the future of the Saga UI, but migrating users from familiar Delphi/WPF workflows to Angular-based replacements is a persistent challenge. Users may resist change or run into friction when features don't yet reach parity.

**Government trust and vendor lock-in concerns.** The Icelandic government has expressed concerns about dependency on Saga as a critical healthcare platform, the security of national health data, and the risk of vendor lock-in to Helix. These concerns must be addressed proactively through transparency, data portability, and standards adoption.

---

## 3. Product Pillars

Future Saga development is organized around four strategic pillars. These pillars represent the core value Saga delivers to healthcare providers and define where the team's product investment is focused over the next 1–2 years.

### 3.1 Registration — Sheets Platform (SagaPlus)

**Goal:** Replace legacy registration workflows with the new Sheets registration platform built in SagaPlus, establishing it as the standard for all patient and entity registration in Saga.

Sheets is the first major module to fully embody the SagaPlus architecture — Angular frontend, modern .NET backend, and a flexible data-driven design. It serves as both a product milestone and a proof point for the modernization strategy.

Key focus areas:

- **Complete feature parity** with legacy Delphi/WPF registration for the workflows being migrated, ensuring clinics can fully transition without regression.
- **Extensible form engine** — Sheets should support configurable registration forms that can adapt to different clinic types, specialties, and future Nordic market requirements without code changes.
- **Data quality and validation** — registration is the entry point for all patient data in Saga. Invest in robust validation, duplicate detection, and integration with national registries (Þjóðskrá) to ensure data quality at the source.
- **Migration and rollout** — use feature flags and gradual rollout (see Section 6) to transition clinics from legacy registration to Sheets, with clear rollback capability.

Sheets also serves as the template for how future SagaPlus modules should be built — its architecture, testing patterns, and deployment approach should be documented and reused.

### 3.2 Journal — Unified Clinical Overview

**Goal:** Build a new Journal overview and search module that aggregates clinical information from all parts of the Saga system — and from external sources — into a single, unified view for clinicians.

The Journal module is the most clinically impactful pillar. Today, clinicians must navigate multiple legacy views and subsystems to piece together a patient's clinical picture. The new Journal will bring this information together.

Key focus areas:

- **Aggregation layer** — the Journal must pull data from existing Saga components (TextView, PatientIndex, and other modules) as well as external systems. This requires a well-defined internal API surface for each data source, even where the underlying implementations remain legacy.
- **Search and filtering** — clinicians need to quickly find relevant entries across a patient's full history. Invest in performant full-text search across journal entries, documents, lab results, and notes, with filtering by date, type, author, and specialty.
- **Read-first, write-later** — prioritize the read/overview experience first. Getting clinicians to adopt the Journal as their primary patient overview is the critical milestone. Writing and documentation workflows can follow once the view is established.
- **External data integration** — the Journal is a natural consumer of FHIR-based data from external systems (lab results, imaging reports, documents from other providers). Align the Journal's data model with the FHIR facade (see Section 5.2) so that internal and external data can be presented uniformly.
- **Performance** — aggregating data from many sources into a single view creates performance risk. Design the aggregation layer with caching, lazy loading, and progressive rendering so the Journal feels fast even when pulling from slow or large data sources.

The Journal module will be the strongest driver of SagaPlus adoption — if clinicians find it genuinely better than the legacy views, it becomes the pull that accelerates the broader modernization.

### 3.3 Billing — B2B with Sjúkratryggingar Íslands (SÍ)

**Goal:** Streamline the billing experience for clinics by focusing on the B2B relationship with Sjúkratryggingar Íslands (SÍ, Icelandic Health Insurance), making claims submission and reimbursement smoother and more automated.

Billing is a pain point for many clinics — the process involves manual steps, error-prone data entry, and slow feedback loops on claim status. Improving this directly impacts clinic operations and revenue.

Key focus areas:

- **Automated claims workflow** — reduce manual steps in claims submission to SÍ. Automate validation of claims data before submission to catch errors early and reduce rejections.
- **Real-time status feedback** — provide clinics with clear visibility into claim status (submitted, accepted, rejected, paid) without requiring them to check external systems or wait for batch reports.
- **Error handling and resubmission** — make it easy for clinic staff to understand why a claim was rejected and to correct and resubmit without starting from scratch.
- **Integration modernization** — if the current SÍ integration is based on legacy protocols or formats, evaluate modernizing to API-based exchange. Align with the broader FHIR strategy where applicable (e.g., FHIR Claim resources), though pragmatism should prevail — if SÍ has a well-defined API that isn't FHIR-based, use it directly.
- **Reporting and analytics** — give clinics summary views of billing performance: submission volumes, rejection rates, average reimbursement times. This helps clinics optimize their billing processes.

### 3.4 Scheduling & Patient Engagement — Silva + Ísland.is

**Goal:** Replace legacy scheduling in Saga with Silva as the primary scheduling platform, while cooperating with the national Ísland.is portal for government-facing patient engagement that extends beyond scheduling.

**Silva** is a separate product developed by Helix that serves as Saga's modern scheduling platform. The strategic direction is to integrate Silva deeply into the Saga workflow, progressively replacing the legacy scheduling modules currently embedded in the Saga desktop client.

**Ísland.is** is the national digital services portal — its role is complementary to Silva, handling government-facing patient engagement functions such as referrals, drug renewal requests, and other interactions that belong in the public health infrastructure rather than in a clinical scheduling tool.

Key focus areas:

- **Silva as scheduling replacement.** Migrate scheduling workflows from legacy Saga modules to Silva. This means tight, reliable synchronization between Saga's clinical data and Silva's scheduling engine — appointments, cancellations, and schedule changes must flow in near-real-time. Define a clear API contract between the two systems and treat Silva integration as a first-class product pillar, not a sidecar.
- **Saga ↔ Silva workflow integration.** Silva should feel like a natural part of the Saga experience for clinic staff, not a separate tool. Invest in the integration layer so that clinicians can manage scheduling from within Saga's UI (whether in the legacy shell or the new multi-platform shell) without context-switching to a separate application.
- **Ísland.is cooperation for patient engagement.** Work with the Ísland.is platform to enable patient-facing services that sit naturally in the government portal: referrals, prescription renewals, health data access, and other functions that patients expect to find alongside their other government digital services. This requires conforming to Ísland.is technical standards, authentication (Auðkenni/electronic ID), and UX guidelines.
- **Clear boundary between Silva and Ísland.is.** Silva handles clinical scheduling and clinic-facing booking workflows. Ísland.is handles government-facing patient engagement. Some functions may touch both (e.g., a patient booking an appointment could flow through Ísland.is to Silva), but the division of responsibility should be explicit and well-documented.
- **Waitlist and capacity management** — give clinics tools within Silva to manage waitlists, optimize appointment slot utilization, and handle overbooking scenarios gracefully.
- **EHDS alignment** — as EHDS requires patients to have access to and control over their electronic health data, the Ísland.is patient engagement layer is a natural channel for fulfilling some of these patient rights. Keep this in mind when designing data access capabilities through the national portal.

---

## 4. Data Sovereignty, Transparency & Government Trust

The Icelandic government has legitimate concerns about vendor lock-in to the Saga platform and about the security and stewardship of health data for the Icelandic population. As Saga consolidates into fewer, larger instances and becomes more deeply embedded in the national healthcare infrastructure, these concerns will only grow. Addressing them proactively is not just good policy — it is a competitive necessity. A government that doesn't trust its healthcare platform will eventually seek alternatives.

Helix's strategy is to meet these concerns through three reinforcing commitments: **data ownership stays with institutions**, **code transparency for government oversight**, and **standards-based interoperability that prevents lock-in**.

### 4.1 Institutional Data Ownership

**Principle: Helix does not own any health data. All data is owned by the healthcare institutions that generate it.**

This is not just a contractual position — it should be architecturally enforced and verifiable:

- **Data portability.** Institutions must be able to export their complete data at any time, in standard formats, without depending on Helix's cooperation or tooling. The FHIR facade (Section 5.2) directly supports this — as FHIR resource coverage grows, so does the institution's ability to extract their data in an internationally recognized format.
- **No data hostage scenarios.** If an institution chooses to leave Saga, their data must be fully extractable. Invest in comprehensive export tooling and document the data model so that institutions (or their chosen vendors) can migrate away. This may seem counterintuitive, but making exit easy actually builds trust and reduces the pressure to mandate alternatives.
- **Clear data processing agreements.** Ensure that contracts and data processing agreements explicitly state that Helix acts as a data processor, not a data controller. Institutions retain full control over their data, including decisions about secondary use under EHDS.
- **Transparent data handling.** Provide institutions with audit dashboards showing who accessed their data, when, and for what purpose. This extends the audit logging capabilities in Section 5.4 into a trust-building tool.

### 4.2 Open Source Strategy

**Principle: While Saga remains a licensed commercial product, the codebase for the newest products is made available to the government for inspection, audit, and long-term security assurance.**

This is a "source-available" or "government open source" model — not full open source in the traditional sense, but a transparency commitment that directly addresses lock-in and security concerns.

**Proposed approach:**

- **SagaPlus modules and new shell — source available to government.** The newest components (Sheets, Journal, Billing modules, the new multi-platform shell, and the FHIR facade) should have their source code accessible to the Icelandic government and its designated auditors. This allows the government to verify security practices, audit data handling, and have confidence that they are not dependent on a black box.
- **Backend services — source available to government.** Modern .NET backend services should follow the same transparency model. The government should be able to inspect how data is processed, stored, and transmitted.
- **Legacy Delphi/WPF — not in scope.** The legacy codebase is being phased out and does not need to be part of the open source strategy. Focus transparency efforts on the components that represent the future.
- **Licensing model.** Consider a dual-license or source-available license (e.g., Business Source License, Server Side Public License, or a custom government inspection license). The goal is to allow government audit and reduce lock-in concerns while protecting Helix's commercial interests. The exact license model requires legal input, but the technical decision to structure the codebase for openness can begin now.
- **Escrow as a minimum.** If full source availability is not immediately feasible, establish a source code escrow arrangement where the government has access to the codebase if Helix ceases operations or fails to maintain the product. This provides a safety net while the broader transparency strategy is developed.

**What this requires technically:**

- **Clean separation of proprietary and open components.** The codebase must be structured so that source-available components don't depend on proprietary libraries or services that would make the code useless without Helix. SagaPlus modules should be self-contained and buildable with standard open-source tooling.
- **Public or government-accessible repositories.** Set up source repositories that government auditors can access. This may be a private GitHub/GitLab instance shared with designated reviewers, or a formal government audit portal.
- **Documentation for external readers.** Source code without context is hard to audit. Invest in architecture documentation, data flow diagrams, and security design documents that allow external reviewers to understand the system without requiring Helix engineers to walk them through it.
- **CI/CD transparency.** Consider making the build and deployment pipeline inspectable, so the government can verify that what runs in production matches the source code they can see.

### 4.3 Standards-Based Interoperability as Anti-Lock-In

The strongest antidote to vendor lock-in is interoperability. If Saga exposes data and functionality through standard interfaces (FHIR, EHDS-compliant formats, standard APIs), then institutions can integrate with or migrate to alternative systems without being trapped by proprietary data formats.

This connects directly to the FHIR and EHDS work in Section 5.2:

- **FHIR as the data liberation layer.** Every FHIR resource Saga exposes is a data pathway that institutions can use independently of Helix. The more complete the FHIR coverage, the less lock-in exists.
- **Standard terminology and coding.** Adopt internationally recognized coding systems (SNOMED CT, ICD-10/11, LOINC) within the FHIR facade. This ensures that exported data is meaningful outside the Saga ecosystem.
- **Open API documentation.** Publish API documentation for the FHIR facade and key integration interfaces. Third-party developers and government technical teams should be able to build against Saga's APIs without proprietary SDKs or Helix involvement.
- **EHDS compliance as proof of openness.** Full EHDS compliance inherently demonstrates that Saga is not a closed system — it can exchange data with any EHDS-compliant system across Europe.

### 4.4 Building the Government Relationship

This strategy is as much about communication and relationship-building as it is about technology:

- **Proactive engagement.** Don't wait for the government to demand transparency — present the data ownership, open source, and interoperability strategy proactively. Frame it as Helix choosing to operate as a trusted partner rather than a locked-in vendor.
- **Regular security audits.** Invite government-designated security auditors to review the Saga codebase and infrastructure, particularly as consolidation increases the system's criticality.
- **EHDS alignment as shared goal.** Position EHDS compliance as a joint initiative between Helix and the Icelandic government. Both parties benefit from Iceland having a compliant, interoperable health data infrastructure.
- **Roadmap transparency.** Share the Saga technical roadmap (this document) with government stakeholders. Demonstrating long-term investment in modernization, security, and standards builds confidence that Saga is a sustainable choice.

---

## 5. Technical Pillars

### 5.1 Architecture & Infrastructure Modernization

**Goal:** Incrementally decouple the monolith so that teams can develop, test, and deploy modules independently — without requiring a full desktop client release cycle.

#### 5.1.1 Strangler Fig Migration

We adopt the strangler fig pattern as our primary modernization strategy. Rather than rewriting the system, we systematically replace Delphi/WPF functionality with SagaPlus Angular modules backed by .NET APIs. Each migration is a deliberate, scoped effort:

1. **Identify** a bounded workflow currently in Delphi/WPF.
2. **Build** the replacement in SagaPlus with full feature parity (or deliberate scope decisions).
3. **Route** users to the new module, keeping the legacy path available during transition.
4. **Retire** the legacy implementation once adoption is confirmed.

A shared prioritization framework should weigh migration candidates by user impact, maintenance cost of the legacy implementation, and integration value. The four product pillars (Registration, Journal, Billing, Scheduling) define the current migration priorities.

#### 5.1.2 Service Boundary Definition

As backend services grow, we need clearer domain boundaries. Over the next year, we should define explicit service contracts for the major Saga domains — aligned with the product pillars: registration, clinical documentation (Journal), billing, and scheduling. This doesn't necessarily mean splitting into microservices — for a team of our size, well-defined modules within a modular monolith or a small number of services is more practical. The key outcome is that each domain has a clear API surface, owns its data, and can be reasoned about independently.

#### 5.1.3 Multi-Tenancy and Instance Consolidation

The ongoing consolidation from many Saga instances to fewer shared deployments requires deliberate architectural support. As organizations are merged into shared instances, the system must cleanly separate tenant data and configuration without creating per-tenant code branches.

- **Tenant-aware data layer.** Ensure all data access includes organization/tenant context. New code should be multi-tenant by default — never assume a single-organization deployment.
- **Configuration over code.** Organization-specific behavior (workflows, form configurations, enabled features, billing rules) should be driven by configuration or feature flags, not by deployment-specific code or branching.
- **Tenant isolation testing.** As part of the CI/CD pipeline, include tests that verify data isolation between tenants — ensuring that one organization's data is never visible to or modifiable by another.
- **Migration tooling.** Build repeatable tooling for migrating an organization from a standalone Saga instance into a consolidated deployment. This includes data migration, configuration mapping, and validation. Each consolidation should be smoother than the last.

#### 5.1.4 Multi-Platform Shell

The current Saga desktop experience is hosted inside the legacy Delphi application, which acts as the shell for all modules — including the newer WPF components and embedded SagaPlus Angular modules. This creates a hard dependency on the Delphi app for all users, even those who primarily use modern modules.

The vision is to build a **new multi-platform shell** that can host SagaPlus modules and third-party systems, freeing Saga from the Delphi dependency while continuing to support legacy Windows-only modules during the transition.

**Design principles for the new shell:**

- **Module host, not monolith.** The shell itself should be thin — its job is to provide navigation, authentication, context management (e.g., current patient, current organization), and a frame for hosting embedded modules. Business logic lives in the modules and their backends, not in the shell.
- **Multi-platform.** The new shell should run on Windows, macOS, and potentially Linux, opening Saga to healthcare organizations that don't use Windows-only environments. This is particularly relevant for Nordic expansion.
- **Third-party extensibility.** Design the module hosting interface to support not just Saga's own SagaPlus modules but also third-party applications. This positions Saga as a platform, not just an application — clinics can integrate best-of-breed tools alongside Saga's core modules.
- **Consistent user experience.** Regardless of which modules are loaded, navigation, patient context, notifications, and session management should feel unified. The shell provides the glue that makes many modules feel like one system.

**Legacy module strategy — three options under evaluation:**

The key architectural question is how the legacy Delphi/WPF modules coexist with the new shell during the transition period. Three approaches are being considered:

**Option A: Embed legacy modules in the new shell (Windows only).** On Windows, the new shell could host the Delphi/WPF components within its process, similar to how the current Delphi shell embeds WPF and Angular modules today. This gives users a single application window but introduces technical complexity — embedding Delphi components in a non-Delphi host requires careful interop work and may limit shell technology choices.

**Option B: Process bridge — two apps, one experience.** The legacy Delphi app continues to run as a separate process alongside the new shell, but the two communicate through a defined interprocess communication (IPC) protocol. Patient context, navigation events, and session state are synchronized so that switching between legacy and modern modules feels seamless to the user. This approach:
- Avoids the technical risk of embedding Delphi components in a new host.
- Allows each application to evolve independently.
- Can be built incrementally — start with basic context sharing (current patient) and expand to deeper integration over time.
- Requires investment in a robust IPC layer (e.g., named pipes, local WebSocket, or a lightweight message bus).

**Option C: Separate shells, gradual migration.** The old Delphi shell remains for legacy modules, and the new shell is used exclusively for SagaPlus modules. Users switch between the two applications manually. This is the simplest to implement but provides the worst user experience during the transition.

The recommendation is to pursue **Option B (process bridge)** as the primary strategy, with the door open to Option A for Windows if the chosen shell technology makes embedding practical. Option C should be the fallback if interop proves too complex, but the goal should be a unified experience.

**Technology evaluation.** The shell technology choice is still being evaluated. Key candidates include:

| Technology | Strengths | Considerations |
|-----------|-----------|----------------|
| **Electron** | Proven for hosting web content, large ecosystem, cross-platform | Memory footprint, distribution size, team needs JavaScript/Node.js experience |
| **Tauri** | Lightweight, Rust-based, uses native webview, smaller footprint than Electron | Younger ecosystem, Rust is a new skillset for the team |
| **.NET MAUI** | Aligns with existing .NET backend skills, native UI per platform | Web module embedding is less natural, cross-platform maturity still evolving |
| **Progressive Web App (PWA)** | No install required, truly cross-platform, aligns with Angular SagaPlus | Limited OS integration, no process bridge for legacy modules, offline capability constraints |

The evaluation should weigh: cross-platform support, ease of embedding Angular web modules, feasibility of IPC with the legacy Delphi app, team skillset, and long-term maintenance cost. A proof-of-concept should be built early to validate the chosen technology against the process bridge requirement.

#### 5.1.5 Deployment Independence

Decouple the release cycles of SagaPlus modules from the desktop client where possible. SagaPlus modules served as embedded web content should be deployable independently, enabling faster iteration on new features without waiting for a full client release. This requires versioned API contracts between the Angular frontend and .NET backends, and a deployment pipeline that supports independent module releases.

#### 5.1.6 Database Strategy

Oracle is deeply entrenched and a full migration is a multi-year effort that may or may not become a priority during this vision's timeframe. Regardless of whether a migration happens, we should make decisions now that reduce coupling to Oracle and keep the door open for PostgreSQL:

- **Abstract data access.** Ensure that new service code accesses data through repository patterns or an ORM (e.g., Entity Framework Core) rather than raw Oracle-specific SQL. Avoid introducing new PL/SQL procedures or Oracle-specific features where a portable alternative exists.
- **Identify Oracle dependencies.** Catalog existing Oracle-specific features in use (PL/SQL packages, Oracle-specific types, materialized views, sequences, etc.) to understand the scope of a potential migration.
- **New modules on PostgreSQL.** For genuinely new, greenfield modules with no legacy data dependencies, evaluate whether they can use PostgreSQL from the start. This reduces Oracle licensing scope incrementally and builds team experience with PostgreSQL.
- **Testcontainers with PostgreSQL.** Where possible, use PostgreSQL-backed Testcontainers for integration testing of new services, even if production still uses Oracle. This reduces test environment costs and validates portability.

A full Oracle-to-PostgreSQL migration — if it becomes a strategic decision — would be a major undertaking deserving its own dedicated planning. The goal during this vision period is to stop digging deeper into Oracle lock-in and prepare the groundwork.

#### 5.1.7 Infrastructure

Continue leveraging Kubernetes (K3s) for backend service hosting. Over this period, focus on:

- Standardized Helm charts or Kustomize overlays for consistent deployments across environments.
- Improved observability: structured logging, distributed tracing, and health dashboards so the team can quickly diagnose production issues.
- Environment parity: dev, staging, and production environments should be as similar as possible to reduce "works on my machine" issues.

### 5.2 Integration, Interoperability & EHDS Compliance

**Goal:** Establish a FHIR-based integration layer that enables standardized data exchange, positions Saga for the Nordic market, and prepares for compliance with the European Health Data Space (EHDS) regulation.

#### 5.2.1 The EHDS Imperative

The European Health Data Space Regulation (EU 2025/327) entered into force on 26 March 2025 and is directly relevant to Iceland through the EEA Agreement. This regulation is not optional — it will require fundamental changes to how Saga handles, exposes, and exchanges health data. The key deadlines are:

| Milestone | Date | Saga Impact |
|-----------|------|-------------|
| Regulation enters into force | March 2025 ✅ | Transition period begins |
| Commission adopts implementing acts | March 2027 | Detailed technical rules published — our FHIR and data exchange strategy must be aligned |
| Patient Summaries & ePrescriptions must be exchangeable cross-border | March 2029 | Saga must support the European EHR exchange format for priority data categories |
| Lab results, medical images & discharge reports exchangeable | March 2031 | Second wave of interoperability requirements |

The EHDS mandates that EHR systems align with a European electronic health record exchange format to ensure interoperability. It introduces certification requirements for EHR vendors on interoperability and security compliance. It also establishes a framework for secondary use of health data — meaning Saga may need to support data extraction for research and policy purposes through designated Health Data Access Bodies.

**What this means for Saga today:** Our current lack of standardized interoperability interfaces is not just a technical debt problem — it is an emerging compliance gap. The FHIR investment described below is not merely a nice-to-have for integration convenience; it is a prerequisite for EHDS readiness.

#### 5.2.2 FHIR Facade Strategy

Rather than remodeling Saga's internal data to FHIR, we build a **FHIR facade** — an integration layer that translates between Saga's internal domain model and FHIR resources. This approach:

- Avoids disrupting the existing data model and business logic.
- Provides a standards-compliant external interface for integrations and EHDS compliance.
- Can be built incrementally, one resource type at a time.

**Priority FHIR resources** — aligned with EHDS priority data categories:

| Priority | FHIR Resource | EHDS Category | Target |
|----------|--------------|---------------|--------|
| **Critical** | Patient | Patient Summaries (2029) | Phase 1 |
| **Critical** | MedicationRequest / MedicationDispense | ePrescriptions / eDispensations (2029) | Phase 1–2 |
| **Critical** | Composition (IPS) | International Patient Summary (2029) | Phase 2 |
| **High** | Appointment / Schedule | Booking integrations | Phase 1 |
| **High** | DiagnosticReport, Observation | Lab results (2031) | Phase 2 |
| **High** | ImagingStudy, DocumentReference | Medical images, discharge reports (2031) | Phase 2–3 |
| **Medium** | Encounter, Condition | Clinical data exchange | Phase 3 |

The International Patient Summary (IPS) FHIR profile deserves special attention as it is the likely format for EHDS patient summary exchange. We should evaluate alignment with the HL7 IPS implementation guide early.

#### 5.2.3 Integration Engine

Introduce a lightweight integration engine (or leverage the existing RabbitMQ infrastructure) to handle inbound and outbound FHIR messages. This engine should support:

- **Subscriptions** — allowing external systems to subscribe to events (e.g., new lab results).
- **Transformation** — mapping between Saga internal events and FHIR resource representations.
- **Audit logging** — all integration traffic should be logged for compliance and troubleshooting.
- **Secondary use data extraction** — in preparation for EHDS secondary use requirements, the integration layer should be designed to support bulk data export in standardized formats.

Evaluate whether an open-source FHIR server (such as HAPI FHIR or Firely Server) makes sense as a foundation, or whether a lighter custom facade is more appropriate for the team's capacity. Given EHDS compliance requirements, a mature FHIR server may reduce the surface area we need to build and maintain ourselves.

#### 5.2.4 Nordic Readiness & EHDS Cross-Border Infrastructure

For Nordic expansion and EHDS compliance, the FHIR layer is necessary but not sufficient. We should also track and prepare for:

- **MyHealth@EU** — the cross-border digital infrastructure through which patient summaries and ePrescriptions will be exchanged. Iceland will need a national contact point that Saga must be able to interface with.
- **HealthData@EU** — the infrastructure for secondary use of health data. Understanding how Health Data Access Bodies will request data from EHR systems like Saga is essential for planning our data export capabilities.
- Country-specific FHIR profiles and implementation guides (e.g., Norwegian HL7 FHIR profiles, Swedish base profiles).
- National health ID systems and identity federation across borders.
- Localization infrastructure (language, terminology, regulatory requirements).

Design the FHIR facade to be profile-aware from the start — the same underlying data may need to be expressed using different national profiles depending on the consumer.

### 5.3 Developer Experience & DevOps

**Goal:** Make the development team faster and more confident through better tooling, testing infrastructure, and development workflows.

#### 5.3.1 Testing Strategy

The current testing situation is a significant drag on velocity. We propose a layered approach:

**Unit and integration tests for .NET services.** This is the most tractable area. All new service code should have meaningful test coverage, and critical existing business logic should be backfilled with tests. Use test containers (e.g., Testcontainers for .NET) to spin up database and RabbitMQ instances for integration tests without depending on shared environments.

**SagaPlus (Angular) component and e2e tests.** Establish a standard testing approach for Angular modules — unit tests for components and services, and a small set of Playwright or Cypress end-to-end tests for critical user flows. These e2e tests should run in CI against a containerized backend.

**Legacy layer testing.** The Delphi/WPF layer is difficult to test automatically. Rather than investing heavily in test automation for code that is being migrated away, focus testing effort on the migration seams — ensuring that the new SagaPlus modules produce the same outcomes as the legacy implementations they replace.

#### 5.3.2 CI/CD Pipeline Improvements

- **Automated quality gates** — every pull request should pass linting, unit tests, and build verification before merge.
- **SagaPlus independent deployment pipeline** — support deploying Angular modules without a full client release (see 5.1.5).
- **Environment provisioning** — make it easy to spin up a working Saga environment for testing. Containerize as much of the backend stack as possible so that developers can run a representative environment locally or in CI.

#### 5.3.3 Developer Onboarding and Documentation

With a team of 5–10 developers, knowledge sharing is critical. Invest in:

- **Architecture Decision Records (ADRs)** — document significant technical decisions and their rationale. This builds institutional knowledge and helps new team members understand why things are the way they are.
- **Module migration playbook** — a repeatable guide for migrating a Delphi/WPF workflow to SagaPlus, covering the technical steps, testing approach, and user rollout strategy.
- **Local development setup guide** — a single, maintained document that gets a new developer from zero to a running Saga environment.

#### 5.3.4 Observability

Invest in observability as a first-class concern:

- **Structured logging** across all .NET services using a consistent format (e.g., Serilog with a shared configuration).
- **Distributed tracing** (OpenTelemetry) to follow requests across service boundaries.
- **Centralized dashboards** (Grafana or similar) showing service health, error rates, and key business metrics.
- **Alerting** for critical failures so the team is notified proactively rather than discovering issues through user reports.

### 5.4 Security Hardening

**Goal:** Strengthen the overall security posture of the Saga platform to match the increased risk profile that comes with instance consolidation, EHDS data exchange requirements, and expanding integration surfaces.

As Saga consolidates from many small instances into fewer shared deployments, the security equation changes fundamentally. A compromised distributed instance affected one organization; a compromised consolidated instance could expose data for many. Combined with EHDS mandating new external data exchange interfaces and the Silva/Ísland.is integrations expanding the attack surface, security must become a proactive, continuous investment — not an afterthought.

#### 5.4.1 Authentication and Authorization

- **Centralized identity management.** As multiple organizations share a single deployment, the identity and access control layer must be robust. Evaluate whether the current authentication mechanism is fit for a multi-tenant consolidated environment. Consider adopting or strengthening an OpenID Connect / OAuth 2.0 based identity layer.
- **Role-based and organization-scoped access.** Every action in the system should be authorized both by the user's role and their organizational context. A clinician in Organization A must never be able to access data belonging to Organization B — and this must be enforced at the API level, not just the UI.
- **API authentication for integrations.** As the FHIR facade and Silva integration expose APIs externally, these must be secured with proper token-based authentication, scoped permissions, and rate limiting. No integration endpoint should be reachable without authentication.

#### 5.4.2 Data Protection

- **Encryption at rest and in transit.** Verify that all data is encrypted at rest in Oracle (and PostgreSQL for any new modules) and that all communication between services, clients, and external systems uses TLS. This is a baseline requirement for EHDS compliance.
- **Tenant data isolation.** Beyond logical isolation in the application layer, consider database-level isolation strategies (schema-per-tenant, row-level security) to provide defense-in-depth against cross-tenant data leaks. Test isolation rigorously — a tenant isolation failure in a healthcare system is a critical incident.
- **Data classification.** Catalog the types of data Saga holds (PII, clinical data, billing data) and ensure that each category has appropriate access controls, retention policies, and audit requirements. EHDS secondary use provisions will require Saga to understand what data it holds and how it can be shared.
- **Secrets management.** Ensure API keys, database credentials, and integration tokens are stored in a secrets management solution (e.g., HashiCorp Vault, Kubernetes Secrets with encryption) rather than in configuration files or code.

#### 5.4.3 Audit and Compliance

- **Comprehensive audit logging.** All access to patient data — reads and writes — should be logged with user identity, organizational context, timestamp, and the action performed. This is essential for EHDS compliance, incident investigation, and regulatory audits.
- **Immutable audit trail.** Audit logs must be tamper-resistant. Store them separately from application data, with restricted access and retention policies that meet healthcare regulatory requirements.
- **Access reviews.** Establish a process for periodic review of user access rights, particularly for privileged accounts and cross-organizational access in consolidated deployments.

#### 5.4.4 Application Security

- **Dependency scanning.** Integrate automated vulnerability scanning for third-party dependencies (.NET NuGet packages, Angular npm packages) into the CI pipeline. Known vulnerabilities in dependencies are one of the most common attack vectors.
- **Static analysis and code review.** Adopt static application security testing (SAST) tools for .NET and Angular codebases. Security-sensitive code paths (authentication, authorization, data access, FHIR API endpoints) should require explicit security review.
- **Penetration testing.** As consolidated instances and new integration surfaces (FHIR, Silva, Ísland.is) go into production, schedule regular penetration testing — particularly for the multi-tenant boundaries and external-facing APIs.
- **Input validation and API hardening.** All external-facing APIs (FHIR facade, Silva integration, any Ísland.is-facing endpoints) must enforce strict input validation, rate limiting, and request size limits. The FHIR facade is a particularly sensitive surface given that it exposes clinical data.

#### 5.4.5 Incident Response

- **Incident response plan.** Document a clear incident response process: who is notified, how is the incident contained, how are affected organizations informed, and what is the post-incident review process. With consolidated deployments, the incident response plan must account for multi-organization impact.
- **Security monitoring.** Extend the observability investment (Section 5.3.4) to include security-specific monitoring: failed authentication attempts, unusual data access patterns, API abuse, and privilege escalation attempts. Alerting should distinguish between operational issues and potential security incidents.

---

## 6. User Adoption Strategy (Technical Perspective)

User adoption of SagaPlus modules is not purely a UX or training problem — it has significant technical dimensions that the development team can directly influence:

**Feature flags and gradual rollout.** Implement a feature flag system that allows new SagaPlus modules to be rolled out to specific user groups, clinics, or roles before a full release. This reduces risk and gives the team real-world feedback before committing to a full migration.

**Seamless embedding.** The transition between legacy desktop and embedded Angular modules should be as invisible as possible. Invest in the shell/host integration layer so that navigation, context passing (e.g., current patient), and visual consistency feel native to users. The new multi-platform shell (Section 5.1.4) and process bridge strategy are key enablers here.

**Telemetry.** Instrument SagaPlus modules to understand actual usage patterns — which features are used, where users get stuck, and how performance compares to the legacy equivalent. This data should drive migration prioritization and UX improvements.

**Feedback loops.** Establish a lightweight mechanism for users to report issues or friction with new modules directly from within the application. Short feedback cycles help the team iterate quickly during the adoption phase.

---

## 7. Prioritized Roadmap

### Phase 1: Foundations (Q2–Q3 2026)

- Define service domain boundaries aligned with product pillars and document them (ADRs).
- Establish CI quality gates and automated test infrastructure (Testcontainers, Angular test setup).
- Set up structured logging and centralized log aggregation across .NET services.
- **Security baseline:** Audit current authentication/authorization model for multi-tenant readiness. Integrate dependency vulnerability scanning into CI. Review encryption at rest and in transit.
- **EHDS gap assessment:** Catalog current integration interfaces and map them against EHDS priority data categories. Identify what Saga can and cannot expose today.
- Begin FHIR facade design: select approach (HAPI/Firely vs. custom), implement Patient and Appointment resources.
- Evaluate HL7 IPS (International Patient Summary) implementation guide for EHDS Patient Summary compliance.
- **Sheets (Registration):** Drive adoption of the Sheets platform, achieve feature parity for priority registration workflows.
- **Journal:** Begin design of the aggregation layer and Journal data model. Define internal API contracts for existing data sources (TextView, PatientIndex).
- Implement feature flag infrastructure for SagaPlus module rollout.
- Create module migration playbook (first draft).
- **New shell:** Evaluate shell technology candidates. Build proof-of-concept for top 1–2 options, testing Angular module hosting and IPC with the Delphi app.
- **Data sovereignty:** Begin structuring codebase for source-available model. Evaluate licensing options.

### Phase 2: Acceleration (Q4 2026 – Q1 2027)

- **Journal:** Deliver the first version of the unified Journal overview — read-only, aggregating key data sources. Begin user rollout.
- **Billing:** Analyze current SÍ integration, design the modernized claims workflow.
- **Silva:** Define and implement the Saga ↔ Silva API contract for scheduling synchronization.
- Deliver first FHIR-based integration (e.g., national patient registry or lab results).
- **FHIR facade for EHDS critical resources:** Patient, MedicationRequest/MedicationDispense, and IPS Composition — aligned with the 2029 EHDS deadline.
- Enable independent deployment of SagaPlus modules.
- Add distributed tracing (OpenTelemetry) and build health dashboards.
- Introduce usage telemetry in SagaPlus modules.
- Engage with Icelandic digital health authority on EHDS implementation timeline and national requirements.
- **Security:** Implement comprehensive audit logging for patient data access. Harden external-facing APIs (FHIR, Silva). Adopt SAST tooling.
- **New shell:** Select shell technology. Implement process bridge IPC protocol (patient context, navigation). Begin internal dogfooding of new shell with early SagaPlus modules.
- **Data sovereignty:** Establish government-accessible source repository for SagaPlus components. Implement institutional data export tooling. Publish open API documentation for FHIR facade.

### Phase 3: Scale (Q2–Q4 2027)

- **Journal:** Add search capabilities, external data integration via FHIR, and expand data source coverage.
- **Billing:** Deliver the improved billing experience with automated validation, real-time status, and reporting.
- **Silva + Ísland.is:** Launch patient self-service scheduling through Ísland.is integration.
- Expand FHIR resource coverage (DiagnosticReport, Observation, ImagingStudy) targeting the 2031 EHDS wave.
- **Secondary use readiness:** Design bulk data export capabilities for Health Data Access Body requests.
- Evaluate MyHealth@EU integration requirements and plan national contact point connectivity.
- Evaluate Nordic expansion requirements: profile-specific FHIR extensions, localization.
- Continue module migrations, prioritized by usage data and integration value.
- Mature observability with alerting and SLA dashboards.
- **Security:** Conduct penetration testing on consolidated instances and external APIs. Establish security monitoring and incident response plan. Implement periodic access reviews.
- **New shell:** Roll out new shell to pilot clinics alongside Delphi app (process bridge). Expand IPC capabilities. Begin onboarding third-party module integrations. Evaluate macOS readiness for Nordic market.
- **Data sovereignty:** Present transparency and data ownership strategy to government stakeholders. Conduct first government-invited security audit.
- Revisit architecture: assess whether further service decomposition is warranted based on team growth and product needs.

---

## 8. Guiding Principles

1. **Incremental over big-bang.** Every change should be deliverable in small, safe steps. No multi-month migrations without intermediate value.
2. **Pragmatic boundaries.** Define clear domain boundaries, but don't over-engineer into microservices prematurely. A modular monolith is fine for a team of our size.
3. **Standards where they matter.** Adopt FHIR for external interoperability and EHDS compliance, but don't force internal systems into FHIR's data model.
4. **Automate the boring stuff.** Invest in CI/CD, test infrastructure, and environment provisioning so the team spends time on product value, not manual processes.
5. **Measure before migrating.** Use telemetry and user feedback to prioritize which legacy modules to replace, rather than guessing.
6. **Document decisions, not just code.** ADRs and playbooks compound in value as the team evolves.
7. **Compliance as enabler, not burden.** Frame EHDS and interoperability work as investments that open markets and integrations, not just regulatory checkboxes.
8. **Earn trust through transparency.** Data ownership, source availability, and standards-based interoperability are not concessions — they are competitive advantages that build lasting government and institutional trust.

---

## 9. Risks and Mitigations

| Risk | Impact | Mitigation |
|------|--------|------------|
| FHIR implementation complexity underestimated | Delays in integration delivery | Start with a minimal facade (2–3 resources), validate with a real integration partner early |
| Legacy module migration takes longer than expected | Prolonged maintenance of dual systems | Strict scoping of migration units; accept that some legacy will persist beyond this horizon |
| Team capacity stretched across modernization and feature work | Neither gets done well | Dedicate a portion of each sprint to infrastructure/modernization work (e.g., 20–30%) rather than treating it as a separate project |
| EHDS compliance timeline is aggressive for current interoperability maturity | Non-compliance, loss of market position | Start with gap assessment immediately; align FHIR work to EHDS priority data categories; engage early with Icelandic digital health authority |
| Oracle licensing costs limit scaling and environments | Expensive test/staging environments, vendor lock-in | Abstract data access in new code, evaluate PostgreSQL for greenfield modules, catalog Oracle-specific dependencies |
| Consolidated instances increase security risk and blast radius | Multi-organization data breach | Security hardening program: multi-tenant access controls, audit logging, penetration testing, dependency scanning, incident response planning |
| Instance consolidation causes data or configuration issues | Disruption for organizations being migrated | Build repeatable migration tooling, run consolidations incrementally, validate data isolation rigorously |
| Nordic expansion requirements are unclear | Architectural rework later | Engage early with potential Nordic partners/customers to understand requirements; design for extensibility |
| User resistance to SagaPlus modules | Low adoption undermines migration ROI | Feature flags, gradual rollout, telemetry-driven iteration, and tight feedback loops |
| Shell technology choice doesn't meet requirements | Rework or technology switch mid-stream | Build proof-of-concept early, test against key requirements (Angular hosting, IPC, cross-platform), make decision before committing to production development |
| Process bridge IPC adds complexity and latency | Jarring user experience between legacy and modern modules | Start with minimal context sharing (patient, session), iterate based on user feedback, set performance budgets for context switches |
| Journal aggregation performance | Slow Journal kills adoption | Design with caching, lazy loading, and progressive rendering from day one; set performance budgets |
| Silva ↔ Saga integration complexity | Scheduling conflicts, data inconsistency | Define clear API contracts early, implement conflict resolution, test with realistic concurrent usage scenarios |
| Government trust erodes despite technical efforts | Political pressure to replace Saga | Engage proactively, deliver on transparency commitments early, treat government relationship as a first-class priority |

---

*This is a living document. It should be revisited quarterly and updated as the team learns from implementation experience and evolving business priorities.*
