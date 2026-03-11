# 4. Data Sovereignty, Transparency & Government Trust

[← Back to Index](README.md) | [Previous: Product Pillars](03-product-pillars.md) | [Next: Architecture Modernization →](05-architecture-modernization.md)

---

The Icelandic government has legitimate concerns about vendor lock-in to the Saga platform and about the security and stewardship of health data for the Icelandic population. As Saga consolidates into fewer, larger instances and becomes more deeply embedded in the national healthcare infrastructure, these concerns will only grow. Addressing them proactively is not just good policy — it is a competitive necessity. A government that doesn't trust its healthcare platform will eventually seek alternatives.

Helix's strategy is to meet these concerns through three reinforcing commitments: **data ownership stays with institutions**, **code transparency for government oversight**, and **standards-based interoperability that prevents lock-in**.

## 4.1 Institutional Data Ownership

**Principle: Helix does not own any health data. All data is owned by the healthcare institutions that generate it.**

This is not just a contractual position — it should be verifiable:

- **Data access through standardized APIs.** Data portability is not a separate workstream — it is a natural consequence of the FHIR and GraphQL APIs we are building for EHDS compliance and third-party integration ([Section 5.2](06-interoperability-ehds.md)). As FHIR resource coverage grows to meet EHDS requirements, institutions gain the ability to access their data in internationally recognized formats through the same interfaces. We do not plan to build dedicated export tooling beyond what these standardized APIs already provide.
- **Clear data processing agreements.** Ensure that contracts and data processing agreements explicitly state that Helix acts as a data processor, not a data controller. Institutions retain full control over their data, including decisions about secondary use under EHDS.
- **Transparent data handling.** Provide institutions with audit dashboards showing who accessed their data, when, and for what purpose. This extends the audit logging capabilities in [Section 5.4](08-security.md) into a trust-building tool.

## 4.2 Open Source Strategy

**Principle: While Saga remains a licensed commercial product, the codebase for the newest products is made available to the government for inspection, audit, and long-term security assurance.**

This is a "source-available" or "government open source" model — not full open source in the traditional sense, but a transparency commitment that directly addresses lock-in and security concerns.

**Proposed approach:**

- **SagaPlus modules and shell — source available to government.** The newest components (Sheets, Journal, Billing modules, the WPF shell, and the FHIR facade) should have their source code accessible to the Icelandic government and its designated auditors. This allows the government to verify security practices, audit data handling, and have confidence that they are not dependent on a black box.
- **Backend services — source available to government.** Modern .NET backend services should follow the same transparency model. The government should be able to inspect how data is processed, stored, and transmitted.
- **Legacy Delphi/WPF — not in scope.** The legacy codebase is being phased out and does not need to be part of the open source strategy. Focus transparency efforts on the components that represent the future.
- **Licensing model.** Consider a dual-license or source-available license (e.g., Business Source License, Server Side Public License, or a custom government inspection license). The goal is to allow government audit and reduce lock-in concerns while protecting Helix's commercial interests. The exact license model requires legal input, but the technical decision to structure the codebase for openness can begin now.
- **Escrow as a minimum.** If full source availability is not immediately feasible, establish a source code escrow arrangement where the government has access to the codebase if Helix ceases operations or fails to maintain the product. This provides a safety net while the broader transparency strategy is developed.

**What this requires technically:**

- **Clean separation of proprietary and open components.** The codebase must be structured so that source-available components don't depend on proprietary libraries or services that would make the code useless without Helix. SagaPlus modules should be self-contained and buildable with standard open-source tooling.
- **Public or government-accessible repositories.** Set up source repositories that government auditors can access. This may be a private GitHub/GitLab instance shared with designated reviewers, or a formal government audit portal.
- **Documentation for external readers.** Source code without context is hard to audit. Invest in architecture documentation, data flow diagrams, and security design documents that allow external reviewers to understand the system without requiring Helix engineers to walk them through it.
- **CI/CD transparency.** Consider making the build and deployment pipeline inspectable, so the government can verify that what runs in production matches the source code they can see.

## 4.3 Standards-Based Interoperability as Anti-Lock-In

The strongest antidote to vendor lock-in is interoperability. When Saga exposes data through standard interfaces — FHIR for EHDS compliance, GraphQL for third-party integration — institutions can access and use their data through well-documented, standards-based APIs rather than being trapped by proprietary formats.

This connects directly to the FHIR and EHDS work in [Section 5.2](06-interoperability-ehds.md):

- **Standards-based access as a byproduct of compliance.** The FHIR endpoints we build for EHDS compliance and the GraphQL API we build for third-party integration naturally provide institutions with standardized access to their data. No separate portability effort is needed — the same APIs that serve integrations and regulatory requirements also serve data access needs.
- **Standard terminology and coding.** Adopt internationally recognized coding systems (SNOMED CT, ICD-10/11, LOINC) within the FHIR facade. This ensures that data exposed through standardized APIs is meaningful outside the Saga ecosystem.
- **Open API documentation.** Publish API documentation for the FHIR facade, GraphQL API, and key integration interfaces. Third-party developers and government technical teams should be able to build against Saga's APIs without proprietary SDKs or Helix involvement.
- **EHDS compliance as proof of openness.** Full EHDS compliance inherently demonstrates that Saga is not a closed system — it can exchange data with any EHDS-compliant system across Europe.

## 4.4 Building the Government Relationship

This strategy is as much about communication and relationship-building as it is about technology:

- **Proactive engagement.** Don't wait for the government to demand transparency — present the data ownership, open source, and interoperability strategy proactively. Frame it as Helix choosing to operate as a trusted partner rather than a locked-in vendor.
- **Regular security audits.** Invite government-designated security auditors to review the Saga codebase and infrastructure, particularly as consolidation increases the system's criticality.
- **EHDS alignment as shared goal.** Position EHDS compliance as a joint initiative between Helix and the Icelandic government. Both parties benefit from Iceland having a compliant, interoperable health data infrastructure.
- **Roadmap transparency.** Share the Saga technical roadmap (this document) with government stakeholders. Demonstrating long-term investment in modernization, security, and standards builds confidence that Saga is a sustainable choice.

---

[← Back to Index](README.md) | [Previous: Product Pillars](03-product-pillars.md) | [Next: Architecture Modernization →](05-architecture-modernization.md)
