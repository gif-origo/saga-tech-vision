# 5.3 Developer Experience & DevOps

[← Back to Index](README.md) | [Previous: Interoperability & EHDS](06-interoperability-ehds.md) | [Next: Security Hardening →](08-security.md)

---

**Goal:** Make the internal development team faster and more confident through better tooling, testing infrastructure, and development workflows — and enable third-party developers to build on the Saga platform through a developer portal, sandbox environments, and well-documented APIs.

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

## 5.3.5 Developer Portal & Third-Party Integration Environment

As Saga evolves from a standalone application into a platform — with a standardized shell module contract ([Section 5.1.4](05-architecture-modernization.md#514-shell-strategy--wpf-modernization--standardized-module-contract)), GraphQL and FHIR APIs ([Section 5.2.5](06-interoperability-ehds.md#525-sagaplus-as-api-platform)), and third-party extensibility as a design goal — we need to invest in the tools and environment that make it practical for external developers to build on Saga.

Without a clear developer experience, third-party integration remains a theoretical capability. The goal is to lower the barrier so that a developer at a partner organization, a government agency, or an independent software vendor can go from "I want to integrate with Saga" to a working prototype with minimal friction and no dependency on Helix engineers.

### Developer Portal

A central, public-facing developer portal that serves as the single entry point for anyone building on the Saga platform:

- **API documentation.** Interactive, up-to-date documentation for the GraphQL API (with schema explorer and query playground) and the FHIR endpoints (with resource definitions and example payloads). Documentation should be generated from the live API schema wherever possible to prevent drift.
- **Shell module contract documentation.** A complete guide to building modules that run inside the Saga shell — the module registration process, context APIs (patient, organization, user session), inter-module navigation, event contracts, and UI guidelines. Include reference implementations and starter templates.
- **Authentication and authorization guide.** Step-by-step instructions for obtaining API credentials, OAuth 2.0 flows, token scoping, and how tenant/organization context works in API calls. Third-party developers need to understand the security model before they write a line of code.
- **Getting started tutorials.** Guided walkthroughs for the most common integration scenarios: "Query patient data via GraphQL," "Build a shell module that displays a custom dashboard," "Subscribe to appointment events via FHIR," "Submit a FHIR resource." These should be copy-paste runnable.
- **Changelog and versioning.** Clear communication of API changes, deprecations, and migration guides. Third-party developers need confidence that their integrations won't break silently.

### Sandbox Environment

A dedicated environment where third-party developers can build and test integrations without touching production data:

- **Self-service provisioning.** Developers should be able to sign up through the portal and get access to a sandbox instance with synthetic (but realistic) data — patients, appointments, journal entries, billing records. No Helix engineer should need to be involved in provisioning.
- **Synthetic data.** The sandbox must contain representative test data that exercises the full API surface — enough variety to test edge cases without exposing any real patient information. Invest in a synthetic data generator that can be refreshed periodically.
- **Isolated tenant.** Each third-party developer (or partner organization) gets their own tenant context in the sandbox, so their test data and configurations don't interfere with other developers.
- **API parity with production.** The sandbox should run the same API versions as production — same GraphQL schema, same FHIR resources, same shell module contract. If it works in sandbox, it should work in production (modulo data and scale).
- **Rate limits and usage monitoring.** Apply rate limits in the sandbox to match production constraints. Provide developers with a dashboard showing their API usage, error rates, and quota.

### Module Development Kit (MDK)

To make third-party module development practical, provide a development kit that abstracts the shell integration complexity:

- **Starter templates.** Project scaffolding for the most common module types — an Angular module that runs in the shell, a standalone web app that uses the GraphQL API, a backend service that consumes FHIR events. Templates should include authentication setup, context handling, and basic navigation integration out of the box.
- **Local development shell.** A lightweight, standalone version of the Saga shell that developers can run locally. It should simulate patient context, navigation events, and session management so that module developers can build and test without deploying to the sandbox. This could be a minimal web-based shell that implements the standardized module contract, or a Docker image that includes the necessary context APIs.
- **CLI tooling.** A command-line tool for common developer tasks: scaffold a new module, validate a module manifest against the shell contract, run the local shell with a module loaded, deploy a module to the sandbox for integration testing.
- **Testing utilities.** Helpers for writing integration tests against the Saga APIs — mock context providers, test fixtures for common data patterns, and a test harness that simulates the shell environment.

### Partner Onboarding and Support

- **Partner program structure.** Define tiers of third-party integration — from lightweight API consumers (read-only data access) to full module developers (shell-hosted applications). Each tier has different requirements for security review, certification, and ongoing support.
- **Module certification.** Before a third-party module can be listed or deployed in production Saga instances, it should pass a certification process covering security (no data leaks, proper authorization), performance (doesn't degrade the shell), and UX consistency (follows shell UI guidelines). The certification criteria should be published and testable — ideally, developers can run a certification check locally before submitting.
- **Developer community channel.** A forum, Slack/Discord workspace, or GitHub Discussions space where third-party developers can ask questions, report issues, and share patterns. This reduces the support burden on the Helix team while building a developer community around the platform.

---

[← Back to Index](README.md) | [Previous: Interoperability & EHDS](06-interoperability-ehds.md) | [Next: Security Hardening →](08-security.md)
