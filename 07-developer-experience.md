# 5.3 Developer Experience & DevOps

[← Back to Index](README.md) | [Previous: Interoperability & EHDS](06-interoperability-ehds.md) | [Next: Security Hardening →](08-security.md)

---

**Goal:** Make the development team faster and more confident through better tooling, testing infrastructure, and development workflows.

## 5.3.1 Testing Strategy

The current testing situation is a significant drag on velocity. We propose a layered approach:

**Unit and integration tests for .NET services.** This is the most tractable area. All new service code should have meaningful test coverage, and critical existing business logic should be backfilled with tests. Use test containers (e.g., Testcontainers for .NET) to spin up database and RabbitMQ instances for integration tests without depending on shared environments.

**SagaPlus (Angular) component and e2e tests.** Establish a standard testing approach for Angular modules — unit tests for components and services, and a small set of Playwright or Cypress end-to-end tests for critical user flows. These e2e tests should run in CI against a containerized backend.

**Legacy layer testing.** The Delphi/WPF layer is difficult to test automatically. Rather than investing heavily in test automation for code that is being migrated away, focus testing effort on the migration seams — ensuring that the new SagaPlus modules produce the same outcomes as the legacy implementations they replace.

## 5.3.2 CI/CD Pipeline Improvements

- **Automated quality gates** — every pull request should pass linting, unit tests, and build verification before merge.
- **SagaPlus independent deployment pipeline** — support deploying Angular modules without a full client release (see [Deployment Independence](05-architecture-modernization.md#515-deployment-independence)).
- **Environment provisioning** — make it easy to spin up a working Saga environment for testing. Containerize as much of the backend stack as possible so that developers can run a representative environment locally or in CI.

## 5.3.3 Developer Onboarding and Documentation

With a team of 5–10 developers, knowledge sharing is critical. Invest in:

- **Architecture Decision Records (ADRs)** — document significant technical decisions and their rationale. This builds institutional knowledge and helps new team members understand why things are the way they are.
- **Module migration playbook** — a repeatable guide for migrating a Delphi/WPF workflow to SagaPlus, covering the technical steps, testing approach, and user rollout strategy.
- **Local development setup guide** — a single, maintained document that gets a new developer from zero to a running Saga environment.

## 5.3.4 Observability

Invest in observability as a first-class concern:

- **Structured logging** across all .NET services using a consistent format (e.g., Serilog with a shared configuration).
- **Distributed tracing** (OpenTelemetry) to follow requests across service boundaries.
- **Centralized dashboards** (Grafana or similar) showing service health, error rates, and key business metrics.
- **Alerting** for critical failures so the team is notified proactively rather than discovering issues through user reports.

---

[← Back to Index](README.md) | [Previous: Interoperability & EHDS](06-interoperability-ehds.md) | [Next: Security Hardening →](08-security.md)
