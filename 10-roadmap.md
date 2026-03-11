# 7. Prioritized Roadmap

[← Back to Index](README.md) | [Previous: User Adoption Strategy](09-user-adoption.md) | [Next: Guiding Principles →](11-guiding-principles.md)

---

## Phase 1: Foundations (Q2–Q3 2026)

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
- **Shell:** Define the standardized module contract (registration, context injection, navigation, events). Prototype SagaPlus module hosting in the WPF shell via WebView2.
- **Developer portal:** Define the shell module contract and GraphQL schema design principles. Begin drafting API documentation structure.
- **Data sovereignty:** Begin structuring codebase for source-available model. Evaluate licensing options.

## Phase 2: Acceleration (Q4 2026 – Q1 2027)

- **Journal:** Deliver the first version of the unified Journal overview — read-only, aggregating key data sources. Begin user rollout.
- **Billing:** Analyze current SÍ integration, design the modernized claims workflow.
- **Silva:** Define and implement the Saga ↔ Silva API contract for patient engagement workflows, starting with scheduling synchronization.
- Deliver first FHIR-based integration (e.g., national patient registry or lab results).
- **FHIR facade for EHDS critical resources:** Patient, MedicationRequest/MedicationDispense, and IPS Composition — aligned with the 2029 EHDS deadline.
- Enable independent deployment of SagaPlus modules.
- Add distributed tracing (OpenTelemetry) and build health dashboards.
- Introduce usage telemetry in SagaPlus modules.
- Engage with Icelandic digital health authority on EHDS implementation timeline and national requirements.
- **Security:** Implement comprehensive audit logging for patient data access. Harden external-facing APIs (FHIR, Silva). Adopt SAST tooling.
- **Shell:** Implement module contract in the WPF shell. Enable SagaPlus modules (Sheets, Journal) to run inside WPF with shared patient context. Begin transitioning users off the Delphi shell for workflows that are covered by WPF + SagaPlus.
- **Developer portal:** Launch developer portal with GraphQL playground, FHIR endpoint documentation, and authentication guide. Deploy sandbox environment with synthetic data. Publish first module starter templates.
- **Data sovereignty:** Establish government-accessible source repository for SagaPlus components. Publish open API documentation for FHIR facade.

## Phase 3: Scale (Q2–Q4 2027)

- **Journal:** Add search capabilities, external data integration via FHIR, and expand data source coverage.
- **Billing:** Deliver the improved billing experience with automated validation, real-time status, and reporting.
- **Silva + Ísland.is:** Launch patient self-service scheduling and engagement workflows through Silva and Ísland.is integration.
- Expand FHIR resource coverage (DiagnosticReport, Observation, ImagingStudy) targeting the 2031 EHDS wave.
- **Secondary use readiness:** Design bulk data export capabilities for Health Data Access Body requests.
- Evaluate MyHealth@EU integration requirements and plan national contact point connectivity.
- Evaluate Nordic expansion requirements: profile-specific FHIR extensions, localization.
- Continue module migrations, prioritized by usage data and integration value.
- Mature observability with alerting and SLA dashboards.
- **Security:** Conduct penetration testing on consolidated instances and external APIs. Establish security monitoring and incident response plan. Implement periodic access reviews.
- **Shell:** Roll out WPF shell as primary application for pilot clinics, replacing Delphi for day-to-day use. Onboard first third-party modules via the standardized module contract. Evaluate whether a multi-platform shell is needed for Nordic expansion — the module contract ensures this remains an option without rework.
- **Developer portal:** Release Module Development Kit (MDK) with local shell, CLI tooling, and testing utilities. Onboard first third-party module partners. Establish module certification process.
- **Data sovereignty:** Present transparency and data ownership strategy to government stakeholders. Conduct first government-invited security audit.
- Revisit architecture: assess whether further service decomposition is warranted based on team growth and product needs.

---

[← Back to Index](README.md) | [Previous: User Adoption Strategy](09-user-adoption.md) | [Next: Guiding Principles →](11-guiding-principles.md)
