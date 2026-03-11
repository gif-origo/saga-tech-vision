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

---

[← Back to Index](README.md) | [Previous: Architecture Modernization](05-architecture-modernization.md) | [Next: Developer Experience →](07-developer-experience.md)
