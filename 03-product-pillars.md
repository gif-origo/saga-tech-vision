# 3. Product Pillars

[← Back to Index](README.md) | [Previous: Where We Are Today](02-current-state.md) | [Next: Data Sovereignty →](04-data-sovereignty.md)

---

Future Saga development is organized around four strategic pillars. These pillars represent the areas of **new strategic investment** over the next 1–2 years — but they are not the entirety of the team's work.

Saga includes a broad portfolio of existing modules — MedCard, Antenatal, Careteams (Meðvera), ADT, Blazor web applications, reporting tools, and many others — that serve critical clinical and administrative needs across the health system. These modules will continue to be actively maintained and enhanced with new features as customer needs evolve. The four pillars below do not replace this ongoing work; they represent the areas where we are making focused, strategic bets on the platform's future. The "reuse before build" principle below applies equally to feature requests within existing modules — where possible, extend what exists rather than building something new.

### Reuse Before Build

A cross-cutting principle governs how we approach new feature requests across all pillars: **exhaust existing capabilities before creating new ones.** Saga is a vast system and the team cannot afford to build purpose-built solutions for every request. When a new feature or workflow is needed, the evaluation order should be:

1. **Can it be solved with what we have?** — A new sheet in the Sheets platform, an adjusted configuration, or a combination of existing modules.
2. **Can a third party solve it?** — Is there an external product or service that handles this well and can integrate through our APIs or run as a module in the shell?
3. **Do we need to build something new?** — Only if the above options genuinely don't fit should we invest in a new custom module.

This may sometimes mean sacrificing a degree of polish or feature completeness compared to a purpose-built solution. That trade-off is intentional — a modular, flexible system that the team can sustain is more valuable than a collection of specialized features that become maintenance burdens. The Sheets platform and the third-party integration strategy are the two primary mechanisms for making this work.

## 3.1 Registration — Sheets Platform (SagaPlus)

**Goal:** Replace legacy registration workflows with the Sheets platform built in SagaPlus, and establish Sheets and its configurable form engine as the **default building block** for structured data capture across Saga — not just for registration.

Sheets is the first major module to fully embody the SagaPlus architecture — Angular frontend, modern .NET backend, and a flexible data-driven design. It serves as both a product milestone and a proof point for the modernization strategy. But Sheets is more than a registration replacement — its configurable form engine is a general-purpose tool for structured data entry that can be adapted to a wide range of clinical and administrative workflows.

**Sheets as the first answer to new feature requests.** When a clinic, specialty, or customer requests a new data capture workflow — a new intake form, a specialized assessment, a referral template — the first question should be: can this be built as a new sheet? If the form engine can accommodate the data model and validation rules, a new sheet can be configured and deployed without writing a new module. This approach:

- Reduces the number of distinct modules the team must build and maintain.
- Gives clinics faster turnaround on workflow requests — configuration is faster than development.
- Keeps the UX consistent — users learn one interface for structured data entry, regardless of the specific workflow.
- Scales to Nordic expansion — new market requirements often mean new forms and workflows, which the Sheets form engine can absorb without code changes.

Key focus areas:

- **Complete feature parity** with legacy Delphi/WPF registration for the workflows being migrated, ensuring clinics can fully transition without regression.
- **Form engine as a platform** — invest in the form engine's flexibility: support for a wide range of field types, conditional logic, validation rules, calculated fields, and configurable layouts. The more capable the form engine becomes, the more feature requests it can absorb without custom development.
- **Data quality and validation** — registration is the entry point for all patient data in Saga. Invest in robust validation, duplicate detection, and integration with national registries (Þjóðskrá) to ensure data quality at the source.
- **Migration and rollout** — use feature flags and gradual rollout (see [User Adoption Strategy](09-user-adoption.md)) to transition clinics from legacy registration to Sheets, with clear rollback capability.
- **Extensibility boundaries** — not everything belongs in a sheet. Define clear criteria for when a workflow has outgrown what the form engine can reasonably handle and genuinely needs a dedicated module or a third-party solution. Avoid overloading Sheets to the point where it becomes brittle.

Sheets also serves as the template for how future SagaPlus modules should be built — its architecture, testing patterns, and deployment approach should be documented and reused.

## 3.2 Journal — Unified Clinical Overview

**Goal:** Build a new Journal overview and search module that aggregates clinical information from all parts of the Saga system — and from external sources — into a single, unified view for clinicians.

The Journal module is the most clinically impactful pillar. Today, clinicians must navigate multiple legacy views and subsystems to piece together a patient's clinical picture. The new Journal will bring this information together.

Key focus areas:

- **Aggregation layer** — the Journal must pull data from existing Saga components (TextView, PatientIndex, and other modules) as well as external systems. This requires a well-defined internal API surface for each data source, even where the underlying implementations remain legacy.
- **Search and filtering** — clinicians need to quickly find relevant entries across a patient's full history. Invest in performant full-text search across journal entries, documents, lab results, and notes, with filtering by date, type, author, and specialty.
- **Read-first, write-later** — prioritize the read/overview experience first. Getting clinicians to adopt the Journal as their primary patient overview is the critical milestone. Writing and documentation workflows can follow once the view is established.
- **External data integration** — the Journal is a natural consumer of FHIR-based data from external systems (lab results, imaging reports, documents from other providers). Align the Journal's data model with the FHIR facade (see [Interoperability & EHDS](06-interoperability-ehds.md)) so that internal and external data can be presented uniformly.
- **Performance** — aggregating data from many sources into a single view creates performance risk. Design the aggregation layer with caching, lazy loading, and progressive rendering so the Journal feels fast even when pulling from slow or large data sources.

The Journal module will be the strongest driver of SagaPlus adoption — if clinicians find it genuinely better than the legacy views, it becomes the pull that accelerates the broader modernization.

## 3.3 Billing — B2B with Sjúkratryggingar Íslands (SÍ)

**Goal:** Streamline the billing experience for clinics by focusing on the B2B relationship with Sjúkratryggingar Íslands (SÍ, Icelandic Health Insurance), making claims submission and reimbursement smoother and more automated.

Billing is a pain point for many clinics — the process involves manual steps, error-prone data entry, and slow feedback loops on claim status. Improving this directly impacts clinic operations and revenue.

Key focus areas:

- **Automated claims workflow** — reduce manual steps in claims submission to SÍ. Automate validation of claims data before submission to catch errors early and reduce rejections.
- **Real-time status feedback** — provide clinics with clear visibility into claim status (submitted, accepted, rejected, paid) without requiring them to check external systems or wait for batch reports.
- **Error handling and resubmission** — make it easy for clinic staff to understand why a claim was rejected and to correct and resubmit without starting from scratch.
- **Integration modernization** — if the current SÍ integration is based on legacy protocols or formats, evaluate modernizing to API-based exchange. Align with the broader FHIR strategy where applicable (e.g., FHIR Claim resources), though pragmatism should prevail — if SÍ has a well-defined API that isn't FHIR-based, use it directly.
- **Reporting and analytics** — give clinics summary views of billing performance: submission volumes, rejection rates, average reimbursement times. This helps clinics optimize their billing processes.

## 3.4 Patient Engagement & Scheduling — Silva + Ísland.is

**Goal:** Establish Silva as the patient engagement platform handling workflows between healthcare providers and patients — including scheduling — while cooperating with the national Ísland.is portal for government-facing patient services.

**Silva** is a separate product developed by Helix that serves as Saga's patient engagement platform. Silva handles the workflows that sit between the clinic and the patient: scheduling appointments, managing waitlists, sending reminders, handling cancellations and rebookings, and other patient-facing interactions that extend beyond what a clinical EHR should own. Scheduling is a core capability within Silva, but it is not the only one — Silva is the natural home for any workflow where the patient is an active participant. The strategic direction is to integrate Silva deeply into the Saga workflow, progressively replacing the legacy scheduling modules currently embedded in the Saga desktop client, and expanding patient engagement capabilities over time.

**Ísland.is** is the national digital services portal — its role is complementary to Silva, handling government-facing patient services such as referrals, drug renewal requests, and other interactions that belong in the public health infrastructure rather than in a clinical or provider-managed tool.

Key focus areas:

- **Silva as the patient engagement layer.** Silva owns the patient-facing side of healthcare workflows. This starts with scheduling — migrating appointment booking, cancellations, and schedule management from legacy Saga modules to Silva — but extends to any workflow where the patient needs to interact with the healthcare provider: appointment reminders, pre-visit questionnaires, follow-up notifications, and similar engagement touchpoints.
- **Scheduling migration.** Migrate scheduling workflows from legacy Saga modules to Silva. This means tight, reliable synchronization between Saga's clinical data and Silva's workflow engine — appointments, cancellations, and schedule changes must flow in near-real-time. Define a clear API contract between the two systems and treat Silva integration as a first-class product pillar, not a sidecar.
- **Saga ↔ Silva workflow integration.** Silva should feel like a natural part of the Saga experience for clinic staff, not a separate tool. Invest in the integration layer so that clinicians can manage scheduling and patient engagement from within Saga's UI (whether in the WPF shell or via browser) without context-switching to a separate application.
- **Ísland.is cooperation for government-facing services.** Work with the Ísland.is platform to enable patient-facing services that sit naturally in the government portal: referrals, prescription renewals, health data access, and other functions that patients expect to find alongside their other government digital services. This requires conforming to Ísland.is technical standards, authentication (Auðkenni/electronic ID), and UX guidelines.
- **Clear boundary between Silva and Ísland.is.** Silva handles provider-to-patient engagement workflows — scheduling, reminders, waitlists, and similar interactions that the healthcare organization manages. Ísland.is handles government-facing patient services — referrals, prescriptions, and health data access that belong in the public infrastructure. Some functions may touch both (e.g., a patient booking an appointment could flow through Ísland.is to Silva), but the division of responsibility should be explicit and well-documented.
- **Waitlist and capacity management** — give clinics tools within Silva to manage waitlists, optimize appointment slot utilization, and handle overbooking scenarios gracefully.
- **EHDS alignment** — as EHDS requires patients to have access to and control over their electronic health data, the Ísland.is patient engagement layer is a natural channel for fulfilling some of these patient rights. Keep this in mind when designing data access capabilities through the national portal.

## 3.5 Reporting & Data Access

**Goal:** Shift reporting from a developer-dependent, custom-built model to one where users can be more self-sufficient in creating reports and accessing the data they need — supplemented by third-party tools that specialize in reporting and analytics.

Reporting is a recurring source of feature requests and development effort. Today, many reporting needs require developer involvement — building custom queries, creating new report views, or modifying existing ones. This doesn't scale, and it pulls development capacity away from product work. The strategic direction is to enable users to serve themselves wherever practical, and to partner with specialized third-party reporting tools rather than building a full reporting platform inside Saga.

### Principles

- **Self-service first.** Users — clinic administrators, managers, quality officers — should be able to create, customize, and run common reports without filing a development request. This means exposing data in a structured, queryable way and providing tools that non-technical users can work with.
- **Saga provides the data, not the reporting engine.** Rather than building sophisticated reporting, dashboarding, and analytics capabilities inside Saga, focus on making Saga's data accessible through well-defined interfaces (GraphQL API, database views, data exports) that third-party reporting tools can consume. Saga's role is to be a good data source, not to compete with purpose-built reporting platforms.
- **Third-party partnerships.** Partner with reporting and analytics vendors that specialize in healthcare data. Tools like Power BI, Metabase, or domain-specific healthcare analytics platforms can connect to Saga's data layer and provide capabilities — ad-hoc queries, dashboards, scheduled reports, data visualization — that would take significant effort to build in-house. This aligns with the "reuse, integrate, then build" principle.

### Key Focus Areas

- **Reporting data layer.** Create a structured data access layer designed for reporting use cases — this could be read-only database views, a reporting-specific API, or a data warehouse/mart that aggregates data from across Saga's modules. The key requirement is that reporting queries don't impact transactional performance and that the data model is understandable by non-developers.
- **GraphQL as a reporting interface.** The GraphQL API ([Section 5.2.5](06-interoperability-ehds.md#525-sagaplus-as-api-platform)) can serve as a flexible data access layer for lighter reporting needs — third-party tools and custom dashboards can query exactly the data they need without requiring direct database access.
- **Standard report templates.** For the most common reporting needs (patient volumes, appointment statistics, billing summaries, quality indicators), provide pre-built report templates that users can run and customize with basic parameters (date range, department, provider). These reduce the barrier to entry for self-service.
- **Third-party tool integration.** Ensure that at least one third-party reporting tool can connect to Saga's data with a supported, documented integration path. This includes authentication, data access permissions (respecting tenant isolation and role-based access), and documentation of the available data model.
- **Deprecate custom report development.** Over time, new reporting requests should be directed toward self-service tools and third-party integrations rather than custom-built report modules. Existing custom reports continue to be maintained, but the default answer to "I need a new report" should be "Can you build it yourself with the tools we provide?" before it becomes a development task.

---

[← Back to Index](README.md) | [Previous: Where We Are Today](02-current-state.md) | [Next: Data Sovereignty →](04-data-sovereignty.md)
