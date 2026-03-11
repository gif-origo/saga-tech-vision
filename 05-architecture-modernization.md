# 5.1 Architecture & Infrastructure Modernization

[← Back to Index](README.md) | [Previous: Data Sovereignty](04-data-sovereignty.md) | [Next: Interoperability & EHDS →](06-interoperability-ehds.md)

---

**Goal:** Incrementally decouple the monolith so that teams can develop, test, and deploy modules independently — without requiring a full desktop client release cycle.

## 5.1.1 Strangler Fig Migration

We adopt the strangler fig pattern as our modernization strategy for the areas where migration is justified. Rather than rewriting the system, we selectively replace Delphi/WPF functionality with modern alternatives where the return on investment is clear.

**Important context: most existing modules continue as-is.** Saga has a large portfolio of specialized modules — MedCard, Antenatal, Careteams (Meðvera), ADT, Blazor web applications, reporting tools, and many others — that are actively used and will continue to be maintained and enhanced. The strangler fig pattern applies to workflows targeted for migration to SagaPlus, not to the entire system. Existing modules that are stable, actively developed, and serving their users well do not need to be migrated for the sake of migration. Feature development and improvements to these modules are ongoing work that exists alongside — not in competition with — the strategic pillars.

When a workflow **is** targeted for migration, each migration is a deliberate, scoped effort — but "migration" does not always mean "build a new custom module." For each legacy workflow, the team should evaluate the replacement approach in this order:

1. **Identify** a bounded workflow currently in Delphi/WPF that is a candidate for migration.
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

## 5.1.4 Shell Strategy — WPF Modernization & Standardized Module Contract

The current Saga desktop experience is hosted inside the legacy Delphi application, which acts as the shell for all modules — including the newer WPF components and embedded SagaPlus Angular modules. This creates a hard dependency on the Delphi app for all users, even those who primarily use modern modules.

SagaPlus modules can already be opened directly in a browser by navigating to a URL. This works well for simpler, self-contained workflows — a user doing a single registration task can open Sheets in a browser tab and get the job done. However, this approach breaks down for the multi-module workflows that dominate real clinical use: a clinician switching between the patient journal, scheduling, billing, and third-party tools throughout their day needs shared context (which patient, which encounter), unified navigation, and seamless transitions between modules. Opening each module in a separate browser tab loses all of that — there is no shared session, no coordinated patient context, and no way to integrate non-web tools.

The shell strategy therefore serves two purposes:

1. **Simplify end-user workflows** by providing a single application that manages patient context, navigation, and session state across all modules — so clinicians can move between registration, journal, scheduling, and billing without losing context or switching between disconnected browser tabs.
2. **Standardize module integration** by defining a clear module hosting contract — how modules register themselves, receive context (current patient, organization, user), communicate with each other, and present a consistent navigation experience. This contract applies equally to Saga's own SagaPlus modules and to third-party modules, positioning Saga as an open platform rather than a closed application.

### Near-Term Focus: Improve the WPF Shell, Leave Delphi

Rather than building a new multi-platform shell from scratch, the immediate focus is to **invest in the existing WPF shell** as the primary module host and transition away from the Delphi shell. The WPF shell already exists as part of the Saga desktop client — the goal is to make it capable of hosting SagaPlus Angular modules and acting as the main application frame, eliminating the dependency on Delphi.

This is a pragmatic choice:

- **WPF is known technology.** The team has existing WPF expertise. Improving what we have is faster and lower risk than evaluating and adopting a new shell technology.
- **Delphi is the bottleneck, not WPF.** The primary pain point is the Delphi dependency — its difficulty to modify, test, and deploy, and its role as the mandatory entry point for all users. Moving to WPF as the primary shell removes this bottleneck without requiring a full platform rewrite.
- **SagaPlus embedding in WPF.** WPF can host embedded web content (via WebView2 or similar), enabling it to host SagaPlus Angular modules alongside native WPF components in a single application window.

Key investments for the WPF shell:

- **SagaPlus module hosting.** Enable the WPF shell to load and display SagaPlus Angular modules via embedded web views, with proper context passing (current patient, organization, user session) between the WPF host and the web content.
- **Navigation and context management.** Implement unified navigation so that users can move between WPF-native components and embedded SagaPlus modules without losing patient context or session state.
- **Delphi migration path.** Identify which Delphi-hosted workflows still require the Delphi shell and create a plan to either migrate them to WPF/SagaPlus or provide them through the WPF shell. The goal is to reach a point where the Delphi application is no longer required for day-to-day clinical use.

### Standardized Module Contract

Regardless of the shell technology, the most important investment is defining a **standardized module contract** — a documented API that governs how any module integrates with the shell. This contract is what turns the shell from a monolithic application into a platform.

The module contract covers:

- **Module registration** — how a module declares itself, its navigation entry points, and its capabilities.
- **Context injection** — how the shell provides the current patient, organization, user session, and other context to modules.
- **Inter-module navigation** — how modules request navigation to other modules while preserving context.
- **Event communication** — how modules publish and subscribe to events (e.g., patient changed, appointment booked).
- **UI guidelines** — layout, theming, and interaction patterns that keep the experience consistent across modules.

This contract must be designed to be **shell-agnostic** — it should not depend on WPF-specific APIs or patterns. Modules that conform to the contract should work in any shell implementation that supports it. This is a deliberate architectural decision:

- **Third-party extensibility today.** A well-defined, documented contract enables third-party developers to build modules that integrate with the WPF shell through the developer portal ([Section 5.3.5](07-developer-experience.md#535-developer-portal--third-party-integration-environment)).
- **Multi-platform shell as a future option.** If a multi-platform shell becomes a strategic priority — for example, to support Nordic expansion into non-Windows environments — the standardized module contract means modules built today will work in that future shell without modification. The contract is the investment that keeps this door open without requiring us to build the multi-platform shell now.
- **Browser fallback.** Modules should continue to be accessible via direct URL for simple workflows, testing, and environments where the shell is not available. The module contract should be designed so that modules can detect whether they are running inside the shell or standalone and adapt accordingly (e.g., reading patient context from the shell API vs. requiring manual patient selection).

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
