# 5.1 Architecture & Infrastructure Modernization

[← Back to Index](README.md) | [Previous: Data Sovereignty](04-data-sovereignty.md) | [Next: Interoperability & EHDS →](06-interoperability-ehds.md)

---

**Goal:** Incrementally decouple the monolith so that teams can develop, test, and deploy modules independently — without requiring a full desktop client release cycle.

## 5.1.1 Strangler Fig Migration

We adopt the strangler fig pattern as our primary modernization strategy. Rather than rewriting the system, we systematically replace Delphi/WPF functionality with modern alternatives. Each migration is a deliberate, scoped effort — but "migration" does not always mean "build a new custom module." For each legacy workflow, the team should evaluate the replacement approach in this order:

1. **Identify** a bounded workflow currently in Delphi/WPF.
2. **Evaluate the replacement path:**
   - **Reuse** — can this workflow be replaced by a new sheet in the Sheets platform? Many data capture and registration workflows can be expressed as configurable forms without writing a dedicated module.
   - **Integrate** — can a third-party product handle this workflow, integrated through the shell and Saga's APIs (GraphQL, FHIR)? If a specialized tool already exists and integrates well, prefer it over building from scratch.
   - **Build** — if the workflow genuinely requires custom logic, UI, or data handling that neither Sheets nor a third party can provide, build a new SagaPlus module.
3. **Route** users to the new solution, keeping the legacy path available during transition.
4. **Retire** the legacy implementation once adoption is confirmed.

This evaluation order is critical. The team's instinct may be to build a purpose-built replacement for every legacy screen, but that creates a maintenance burden that grows faster than the team. A sheet in the Sheets platform or a well-integrated third-party tool may cover 80% of a legacy workflow's value at 20% of the development and maintenance cost — and that is often the right trade-off.

A shared prioritization framework should weigh migration candidates by user impact, maintenance cost of the legacy implementation, and integration value. The four product pillars (Registration, Journal, Billing, Scheduling) define the current migration priorities.

## 5.1.2 Service Boundary Definition

As backend services grow, we need clearer domain boundaries. Over the next year, we should define explicit service contracts for the major Saga domains — aligned with the product pillars: registration, clinical documentation (Journal), billing, and scheduling. This doesn't necessarily mean splitting into microservices — for a team of our size, well-defined modules within a modular monolith or a small number of services is more practical. The key outcome is that each domain has a clear API surface, owns its data, and can be reasoned about independently.

The SagaPlus backend already embodies this thinking — each SagaPlus module has its own .NET backend with business logic and data access, rather than being a thin UI over shared legacy services. This makes SagaPlus the natural foundation for a platform API strategy. As we define service boundaries, the SagaPlus backend APIs should be designed not just to serve the Angular frontend but to be consumable by third-party integrations as well. The GraphQL and FHIR API surfaces described in [Section 5.2.5](06-interoperability-ehds.md#525-sagaplus-as-api-platform) build directly on these service boundaries.

## 5.1.3 Multi-Tenancy and Instance Consolidation

The ongoing consolidation from many Saga instances to fewer shared deployments requires deliberate architectural support. As organizations are merged into shared instances, the system must cleanly separate tenant data and configuration without creating per-tenant code branches.

- **Tenant-aware data layer.** Ensure all data access includes organization/tenant context. New code should be multi-tenant by default — never assume a single-organization deployment.
- **Configuration over code.** Organization-specific behavior (workflows, form configurations, enabled features, billing rules) should be driven by configuration or feature flags, not by deployment-specific code or branching.
- **Tenant isolation testing.** As part of the CI/CD pipeline, include tests that verify data isolation between tenants — ensuring that one organization's data is never visible to or modifiable by another.
- **Migration tooling.** Build repeatable tooling for migrating an organization from a standalone Saga instance into a consolidated deployment. This includes data migration, configuration mapping, and validation. Each consolidation should be smoother than the last.

## 5.1.4 Multi-Platform Shell

The current Saga desktop experience is hosted inside the legacy Delphi application, which acts as the shell for all modules — including the newer WPF components and embedded SagaPlus Angular modules. This creates a hard dependency on the Delphi app for all users, even those who primarily use modern modules.

SagaPlus modules can already be opened directly in a browser by navigating to a URL. This works well for simpler, self-contained workflows — a user doing a single registration task can open Sheets in a browser tab and get the job done. However, this approach breaks down for the multi-module workflows that dominate real clinical use: a clinician switching between the patient journal, scheduling, billing, and third-party tools throughout their day needs shared context (which patient, which encounter), unified navigation, and seamless transitions between modules. Opening each module in a separate browser tab loses all of that — there is no shared session, no coordinated patient context, and no way to integrate non-web tools.

The shell strategy therefore serves two purposes:

1. **Simplify end-user workflows** by providing a single application that manages patient context, navigation, and session state across all modules — so clinicians can move between registration, journal, scheduling, and billing without losing context or switching between disconnected browser tabs.
2. **Standardize module integration** by defining a clear module hosting contract — how modules register themselves, receive context (current patient, organization, user), communicate with each other, and present a consistent navigation experience. This contract applies equally to Saga's own SagaPlus modules and to third-party modules, positioning Saga as an open platform rather than a closed application.

The vision is to build a **new multi-platform shell** that fulfills both roles — freeing Saga from the Delphi dependency while continuing to support legacy Windows-only modules during the transition. The browser-accessible mode remains available as a lightweight option for simple workflows and for environments where the shell cannot be installed.

### Design Principles for the New Shell

- **Module host, not monolith.** The shell itself should be thin — its job is to provide navigation, authentication, context management (e.g., current patient, current organization), and a frame for hosting embedded modules. Business logic lives in the modules and their backends, not in the shell.
- **Standardized module contract.** Define a clear, documented API that all modules — first-party and third-party — use to integrate with the shell. This contract covers module registration, context injection (patient, organization, user session), inter-module navigation, and event communication. A well-defined contract lowers the barrier for building new modules and enables third-party developers to integrate without deep knowledge of Saga internals.
- **Multi-platform.** The new shell should run on Windows, macOS, and potentially Linux, opening Saga to healthcare organizations that don't use Windows-only environments. This is particularly relevant for Nordic expansion.
- **Third-party extensibility.** The standardized module contract naturally supports third-party applications. This positions Saga as a platform, not just an application — clinics can integrate best-of-breed tools alongside Saga's core modules.
- **Consistent user experience.** Regardless of which modules are loaded, navigation, patient context, notifications, and session management should feel unified. The shell provides the glue that makes many modules feel like one system.
- **Browser fallback.** Modules should continue to be accessible via direct URL for simple workflows, testing, and environments where the shell is not available. The module contract should be designed so that modules can detect whether they are running inside the shell or standalone and adapt accordingly (e.g., reading patient context from the shell API vs. requiring manual patient selection).

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
