# 5.2 Integration, Interoperability & EHDS Compliance

[← Back to Index](README.md) | [Previous: Architecture Modernization](05-architecture-modernization.md) | [Next: Developer Experience →](07-developer-experience.md)

---

**Goal:** Establish a FHIR-based integration layer that enables standardized data exchange, positions Saga for the Nordic market, and prepares for compliance with the European Health Data Space (EHDS) regulation.

## 5.2.1 The EHDS Imperative

The European Health Data Space Regulation (EU 2025/327) entered into force on 26 March 2025 and is directly relevant to Iceland through the EEA Agreement. This regulation is not optional — it will require fundamental changes to how Saga handles, exposes, and exchanges health data. The key deadlines are:

| Milestone | Date | Saga Impact |
|-----------|------|-------------|
| Regulation enters into force | March 2025 ✅ | Transition period begins |
| Commission adopts implementing acts | March 2027 | Detailed technical rules published — our FHIR and data exchange strategy must be aligned |
| Patient Summaries & ePrescriptions must be exchangeable cross-border | March 2029 | Saga must support the European EHR exchange format for priority data categories |
| Lab results, medical images & discharge reports exchangeable | March 2031 | Second wave of interoperability requirements |

The EHDS mandates that EHR systems align with a European electronic health record exchange format to ensure interoperability. It introduces certification requirements for EHR vendors on interoperability and security compliance. It also establishes a framework for secondary use of health data — meaning Saga may need to support data extraction for research and policy purposes through designated Health Data Access Bodies.

**What this means for Saga today:** Our current lack of standardized interoperability interfaces is not just a technical debt problem — it is an emerging compliance gap. The FHIR investment described below is not merely a nice-to-have for integration convenience; it is a prerequisite for EHDS readiness.

## 5.2.2 FHIR Facade Strategy

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

## 5.2.3 Integration Engine

Introduce a lightweight integration engine (or leverage the existing RabbitMQ infrastructure) to handle inbound and outbound FHIR messages. This engine should support:

- **Subscriptions** — allowing external systems to subscribe to events (e.g., new lab results).
- **Transformation** — mapping between Saga internal events and FHIR resource representations.
- **Audit logging** — all integration traffic should be logged for compliance and troubleshooting.
- **Secondary use data extraction** — in preparation for EHDS secondary use requirements, the integration layer should be designed to support bulk data export in standardized formats.

Evaluate whether an open-source FHIR server (such as HAPI FHIR or Firely Server) makes sense as a foundation, or whether a lighter custom facade is more appropriate for the team's capacity. Given EHDS compliance requirements, a mature FHIR server may reduce the surface area we need to build and maintain ourselves.

## 5.2.4 Nordic Readiness & EHDS Cross-Border Infrastructure

For Nordic expansion and EHDS compliance, the FHIR layer is necessary but not sufficient. We should also track and prepare for:

- **MyHealth@EU** — the cross-border digital infrastructure through which patient summaries and ePrescriptions will be exchanged. Iceland will need a national contact point that Saga must be able to interface with.
- **HealthData@EU** — the infrastructure for secondary use of health data. Understanding how Health Data Access Bodies will request data from EHR systems like Saga is essential for planning our data export capabilities.
- Country-specific FHIR profiles and implementation guides (e.g., Norwegian HL7 FHIR profiles, Swedish base profiles).
- National health ID systems and identity federation across borders.
- Localization infrastructure (language, terminology, regulatory requirements).

Design the FHIR facade to be profile-aware from the start — the same underlying data may need to be expressed using different national profiles depending on the consumer.

## 5.2.5 SagaPlus as API Platform

SagaPlus is not just a UI modernization — its .NET backend services contain real business logic, domain rules, and data access that represent Saga's modern capabilities. This creates an opportunity to position the SagaPlus backend as a **platform API layer** that serves three audiences: the SagaPlus Angular frontend, third-party module developers, and external system integrators.

### GraphQL API

Introduce a **GraphQL interface** on the SagaPlus backend as the primary API for rich, interactive integrations:

- **Flexible querying.** GraphQL lets third-party developers request exactly the data they need — patient demographics, clinical summaries, scheduling slots, billing status — without over-fetching or requiring multiple round trips. This is particularly valuable for module developers building UI-heavy integrations where data needs vary by view.
- **Schema as documentation.** A well-designed GraphQL schema serves as a living, self-documenting API contract. Third-party developers can explore available data and operations through standard tooling (GraphQL Playground, introspection) without relying on separate API documentation that drifts from reality.
- **Module-aligned schema design.** Organize the GraphQL schema along the same domain boundaries as the SagaPlus modules — registration, journal, billing, scheduling. Each module's backend contributes its portion of the schema, keeping the API aligned with the product architecture.
- **Authorization and tenant scoping.** The GraphQL layer must enforce the same role-based and organization-scoped access controls as the rest of the platform (see [Security Hardening](08-security.md)). Every query and mutation must respect the caller's permissions and tenant context.
- **Subscriptions for real-time updates.** GraphQL subscriptions can provide real-time data to third-party modules — e.g., notifying a scheduling widget when an appointment is booked or alerting a dashboard when new lab results arrive.

### FHIR Endpoints on the SagaPlus Backend

The FHIR facade described in Section 5.2.2 should be built on top of the SagaPlus backend services rather than as a separate, disconnected layer:

- **SagaPlus as the FHIR source of truth.** As SagaPlus modules replace legacy workflows, the SagaPlus backend becomes the authoritative source for an increasing share of Saga's data. FHIR endpoints should translate directly from SagaPlus domain models to FHIR resources, rather than reaching back into legacy data stores where possible.
- **Shared business logic.** By building FHIR endpoints on the same backend that serves the UI, we avoid duplicating validation, authorization, and business rules. A FHIR Patient resource and a GraphQL patient query go through the same domain logic.
- **Incremental FHIR coverage follows module migration.** As each product pillar migrates to SagaPlus, the corresponding FHIR resources can be built on the new backend. This naturally aligns FHIR coverage with the modernization roadmap.

### Two APIs, Two Purposes

The GraphQL and FHIR interfaces serve complementary roles:

| | GraphQL | FHIR |
|---|---------|------|
| **Primary audience** | Third-party module developers, internal frontends | External system integrators, cross-border health data exchange |
| **Strength** | Flexible, efficient queries tailored to UI and workflow needs | Standards-based interoperability, EHDS compliance |
| **Schema** | Saga-specific, domain-aligned | HL7 FHIR specification, national profiles |
| **Use cases** | Custom modules in the shell, dashboards, workflow extensions | Lab integrations, national registries, EHDS compliance, cross-border data exchange |

Both APIs are built on the same SagaPlus backend and share the same business logic, authorization, and data access — they are different projections of the same platform, not separate systems.

### What This Enables

- **Third-party module ecosystem.** With a GraphQL API and a standardized shell module contract ([Section 5.1.4](05-architecture-modernization.md#514-multi-platform-shell)), external developers can build modules that run inside the Saga shell and query platform data — without needing access to Saga's internals.
- **Integration without lock-in.** External systems can choose FHIR for standards-based exchange or GraphQL for richer, Saga-specific integrations. Neither requires proprietary SDKs or Helix involvement.
- **Data sovereignty support.** Both APIs naturally support the data ownership principles in [Section 4](04-data-sovereignty.md) — institutions can access their data through standardized interfaces without requiring dedicated export tooling.

---

[← Back to Index](README.md) | [Previous: Architecture Modernization](05-architecture-modernization.md) | [Next: Developer Experience →](07-developer-experience.md)
