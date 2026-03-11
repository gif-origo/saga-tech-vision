# 5.1 Architecture & Infrastructure Modernization

[← Back to Index](README.md) | [Previous: Data Sovereignty](04-data-sovereignty.md) | [Next: Interoperability & EHDS →](06-interoperability-ehds.md)

---

**Goal:** Incrementally decouple the monolith so that teams can develop, test, and deploy modules independently — without requiring a full desktop client release cycle.

## 5.1.1 Strangler Fig Migration

We adopt the strangler fig pattern as our primary modernization strategy. Rather than rewriting the system, we systematically replace Delphi/WPF functionality with SagaPlus Angular modules backed by .NET APIs. Each migration is a deliberate, scoped effort:

1. **Identify** a bounded workflow currently in Delphi/WPF.
2. **Build** the replacement in SagaPlus with full feature parity (or deliberate scope decisions).
3. **Route** users to the new module, keeping the legacy path available during transition.
4. **Retire** the legacy implementation once adoption is confirmed.

A shared prioritization framework should weigh migration candidates by user impact, maintenance cost of the legacy implementation, and integration value. The four product pillars (Registration, Journal, Billing, Scheduling) define the current migration priorities.

## 5.1.2 Service Boundary Definition

As backend services grow, we need clearer domain boundaries. Over the next year, we should define explicit service contracts for the major Saga domains — aligned with the product pillars: registration, clinical documentation (Journal), billing, and scheduling. This doesn't necessarily mean splitting into microservices — for a team of our size, well-defined modules within a modular monolith or a small number of services is more practical. The key outcome is that each domain has a clear API surface, owns its data, and can be reasoned about independently.

## 5.1.3 Multi-Tenancy and Instance Consolidation

The ongoing consolidation from many Saga instances to fewer shared deployments requires deliberate architectural support. As organizations are merged into shared instances, the system must cleanly separate tenant data and configuration without creating per-tenant code branches.

- **Tenant-aware data layer.** Ensure all data access includes organization/tenant context. New code should be multi-tenant by default — never assume a single-organization deployment.
- **Configuration over code.** Organization-specific behavior (workflows, form configurations, enabled features, billing rules) should be driven by configuration or feature flags, not by deployment-specific code or branching.
- **Tenant isolation testing.** As part of the CI/CD pipeline, include tests that verify data isolation between tenants — ensuring that one organization's data is never visible to or modifiable by another.
- **Migration tooling.** Build repeatable tooling for migrating an organization from a standalone Saga instance into a consolidated deployment. This includes data migration, configuration mapping, and validation. Each consolidation should be smoother than the last.

## 5.1.4 Multi-Platform Shell

The current Saga desktop experience is hosted inside the legacy Delphi application, which acts as the shell for all modules — including the newer WPF components and embedded SagaPlus Angular modules. This creates a hard dependency on the Delphi app for all users, even those who primarily use modern modules.

The vision is to build a **new multi-platform shell** that can host SagaPlus modules and third-party systems, freeing Saga from the Delphi dependency while continuing to support legacy Windows-only modules during the transition.

### Design Principles for the New Shell

- **Module host, not monolith.** The shell itself should be thin — its job is to provide navigation, authentication, context management (e.g., current patient, current organization), and a frame for hosting embedded modules. Business logic lives in the modules and their backends, not in the shell.
- **Multi-platform.** The new shell should run on Windows, macOS, and potentially Linux, opening Saga to healthcare organizations that don't use Windows-only environments. This is particularly relevant for Nordic expansion.
- **Third-party extensibility.** Design the module hosting interface to support not just Saga's own SagaPlus modules but also third-party applications. This positions Saga as a platform, not just an application — clinics can integrate best-of-breed tools alongside Saga's core modules.
- **Consistent user experience.** Regardless of which modules are loaded, navigation, patient context, notifications, and session management should feel unified. The shell provides the glue that makes many modules feel like one system.

### Legacy Module Strategy — Three Options Under Evaluation

The key architectural question is how the legacy Delphi/WPF modules coexist with the new shell during the transition period. Three approaches are being considered:

**Option A: Embed legacy modules in the new shell (Windows only).** On Windows, the new shell could host the Delphi/WPF components within its process, similar to how the current Delphi shell embeds WPF and Angular modules today. This gives users a single application window but introduces technical complexity — embedding Delphi components in a non-Delphi host requires careful interop work and may limit shell technology choices.

**Option B: Process bridge — two apps, one experience.** The legacy Delphi app continues to run as a separate process alongside the new shell, but the two communicate through a defined interprocess communication (IPC) protocol. Patient context, navigation events, and session state are synchronized so that switching between legacy and modern modules feels seamless to the user. This approach:
- Avoids the technical risk of embedding Delphi components in a new host.
- Allows each application to evolve independently.
- Can be built incrementally — start with basic context sharing (current patient) and expand to deeper integration over time.
- Requires investment in a robust IPC layer (e.g., named pipes, local WebSocket, or a lightweight message bus).

**Option C: Separate shells, gradual migration.** The old Delphi shell remains for legacy modules, and the new shell is used exclusively for SagaPlus modules. Users switch between the two applications manually. This is the simplest to implement but provides the worst user experience during the transition.

The recommendation is to pursue **Option B (process bridge)** as the primary strategy, with the door open to Option A for Windows if the chosen shell technology makes embedding practical. Option C should be the fallback if interop proves too complex, but the goal should be a unified experience.

### Technology Evaluation

The shell technology choice is still being evaluated. Key candidates include:

| Technology | Strengths | Considerations |
|-----------|-----------|----------------|
| **Electron** | Proven for hosting web content, large ecosystem, cross-platform | Memory footprint, distribution size, team needs JavaScript/Node.js experience |
| **Tauri** | Lightweight, Rust-based, uses native webview, smaller footprint than Electron | Younger ecosystem, Rust is a new skillset for the team |
| **.NET MAUI** | Aligns with existing .NET backend skills, native UI per platform | Web module embedding is less natural, cross-platform maturity still evolving |
| **Progressive Web App (PWA)** | No install required, truly cross-platform, aligns with Angular SagaPlus | Limited OS integration, no process bridge for legacy modules, offline capability constraints |

The evaluation should weigh: cross-platform support, ease of embedding Angular web modules, feasibility of IPC with the legacy Delphi app, team skillset, and long-term maintenance cost. A proof-of-concept should be built early to validate the chosen technology against the process bridge requirement.

## 5.1.5 Deployment Independence

Decouple the release cycles of SagaPlus modules from the desktop client where possible. SagaPlus modules served as embedded web content should be deployable independently, enabling faster iteration on new features without waiting for a full client release. This requires versioned API contracts between the Angular frontend and .NET backends, and a deployment pipeline that supports independent module releases.

## 5.1.6 Database Strategy

Oracle is deeply entrenched and a full migration is a multi-year effort that may or may not become a priority during this vision's timeframe. Regardless of whether a migration happens, we should make decisions now that reduce coupling to Oracle and keep the door open for PostgreSQL:

- **Abstract data access.** Ensure that new service code accesses data through repository patterns or an ORM (e.g., Entity Framework Core) rather than raw Oracle-specific SQL. Avoid introducing new PL/SQL procedures or Oracle-specific features where a portable alternative exists.
- **Identify Oracle dependencies.** Catalog existing Oracle-specific features in use (PL/SQL packages, Oracle-specific types, materialized views, sequences, etc.) to understand the scope of a potential migration.
- **New modules on PostgreSQL.** For genuinely new, greenfield modules with no legacy data dependencies, evaluate whether they can use PostgreSQL from the start. This reduces Oracle licensing scope incrementally and builds team experience with PostgreSQL.
- **Testcontainers with PostgreSQL.** Where possible, use PostgreSQL-backed Testcontainers for integration testing of new services, even if production still uses Oracle. This reduces test environment costs and validates portability.

A full Oracle-to-PostgreSQL migration — if it becomes a strategic decision — would be a major undertaking deserving its own dedicated planning. The goal during this vision period is to stop digging deeper into Oracle lock-in and prepare the groundwork.

## 5.1.7 Infrastructure

Continue leveraging Kubernetes (K3s) for backend service hosting. Over this period, focus on:

- Standardized Helm charts or Kustomize overlays for consistent deployments across environments.
- Improved observability: structured logging, distributed tracing, and health dashboards so the team can quickly diagnose production issues.
- Environment parity: dev, staging, and production environments should be as similar as possible to reduce "works on my machine" issues.

---

[← Back to Index](README.md) | [Previous: Data Sovereignty](04-data-sovereignty.md) | [Next: Interoperability & EHDS →](06-interoperability-ehds.md)
