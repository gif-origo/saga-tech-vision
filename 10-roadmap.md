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
- **New shell:** Evaluate shell technology candidates. Build proof-of-concept for top 1–2 options, testing Angular module hosting and IPC with the Delphi app.
- **Data sovereignty:** Begin structuring codebase for source-available model. Evaluate licensing options.

## Phase 2: Acceleration (Q4 2026 – Q1 2027)

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

## Phase 3: Scale (Q2–Q4 2027)

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

[← Back to Index](README.md) | [Previous: User Adoption Strategy](09-user-adoption.md) | [Next: Guiding Principles →](11-guiding-principles.md)
