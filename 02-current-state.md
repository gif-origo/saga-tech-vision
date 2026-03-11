# 2. Where We Are Today

[← Back to Index](README.md) | [Previous: Executive Summary](01-executive-summary.md) | [Next: Product Pillars →](03-product-pillars.md)

---

## 2.1 Architecture Landscape

Saga's current architecture reflects its history. The system consists of several layers that coexist in production:

- **Delphi desktop client** — the original UI layer and current application shell, still in active use for core workflows. Difficult to modify, test, and deploy. All other UI components (WPF, SagaPlus) are embedded within this shell.
- **WPF components** — a partial modernization of the desktop experience, sharing the client process with the Delphi layer.
- **SagaPlus (Angular)** — embedded web modules delivered inside the desktop shell, representing the newest user-facing functionality. Lives in a separate codebase.
- **Modern .NET backend services** — all backend development uses modern .NET, providing APIs for the SagaPlus modules and handling business logic, data access, and integrations.
- **Oracle Database** — the primary data store. Oracle is deeply embedded in the system and represents a significant operational cost. While there is no immediate migration plan, PostgreSQL is the preferred target if a database transition becomes a strategic priority.
- **RabbitMQ** — used for asynchronous messaging between services.

## 2.2 Deployment Landscape

Saga is currently deployed as multiple distributed instances, each serving different healthcare organizations. This multi-instance model has historically allowed organizations to operate independently, but it comes with significant costs: every deployment must be maintained, upgraded, and monitored separately, and configuration drift between instances creates testing and support complexity.

An ongoing strategic initiative is consolidating these separate deployments — joining organizations into fewer, shared instances. This consolidation represents one of the most impactful opportunities for the Saga team in the near term:

- **Reduced operational overhead.** Fewer instances means fewer deployments to manage, fewer environments to patch, and fewer configurations to keep in sync. This directly frees up developer and operations time for product work.
- **Faster, more confident releases.** With fewer deployment targets, the team can invest in a single, well-tested release pipeline rather than managing variations across many instances. Releases become more predictable and less risky.
- **Consistent feature rollout.** Consolidated instances make it feasible to roll out new SagaPlus modules (Sheets, Journal, Billing) to all organizations simultaneously rather than managing staggered rollouts across dozens of separate deployments.
- **Simplified testing.** Instead of testing against multiple instance configurations, the team can focus on a smaller number of representative environments, improving test coverage and reducing the cost of environment provisioning.
- **Foundation for multi-tenancy.** Consolidation naturally pushes the architecture toward multi-tenant patterns — shared infrastructure with organization-level data isolation, configurable workflows, and role-based access. Investing in proper multi-tenancy now will also support Nordic expansion, where new customers can be onboarded to an existing instance rather than requiring a dedicated deployment.

The consolidation strategy should be treated as an enabler for everything else in this vision. Architectural decisions — particularly around data isolation, configuration management, and deployment pipelines — should assume a consolidated deployment model as the target state. However, consolidation also concentrates risk — a single consolidated instance serving multiple organizations becomes a higher-value target for attackers and a larger blast radius for security incidents. This is addressed in [Section 5.4 — Security Hardening](08-security.md).

## 2.3 Key Challenges

**Integration gaps and EHDS compliance.** Saga currently lacks standardized healthcare interoperability interfaces. Integrations with external systems (labs, imaging, national registries) are largely custom point-to-point connections. This creates maintenance burden and is a barrier to Nordic expansion. More urgently, the European Health Data Space (EHDS) regulation entered into force in March 2025 and will require EHR systems — including Saga — to support standardized data exchange formats by 2029. Our current interoperability gap is an emerging compliance risk.

**Testing and environment management.** The mixed technology stack makes it difficult to set up reliable test environments. The Delphi layer is particularly hard to test in isolation, and end-to-end testing across the full stack requires significant manual effort. Oracle licensing costs further complicate environment provisioning — spinning up multiple test or staging instances is expensive, limiting the team's ability to test in production-like conditions.

**Database cost and lock-in.** Oracle is a considerable operational expense and creates vendor lock-in. Oracle-specific features (PL/SQL procedures, proprietary syntax, specific data types) are likely embedded throughout the codebase, making a future migration non-trivial. While a full database migration is not the current priority, the cost and rigidity of Oracle is a strategic concern that should inform architectural decisions going forward.

**Desktop shell dependency.** All Saga UI components — including modern SagaPlus modules — are currently hosted inside the legacy Delphi application. This ties the entire user experience to a Windows-only, hard-to-maintain shell and blocks multi-platform expansion.

**User adoption of new modules.** SagaPlus modules represent the future of the Saga UI, but migrating users from familiar Delphi/WPF workflows to Angular-based replacements is a persistent challenge. Users may resist change or run into friction when features don't yet reach parity.

**Government trust and vendor lock-in concerns.** The Icelandic government has expressed concerns about dependency on Saga as a critical healthcare platform, the security of national health data, and the risk of vendor lock-in to Helix. These concerns must be addressed proactively through transparency, data portability, and standards adoption.

---

[← Back to Index](README.md) | [Previous: Executive Summary](01-executive-summary.md) | [Next: Product Pillars →](03-product-pillars.md)
