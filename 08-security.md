# 5.4 Security Hardening

[← Back to Index](README.md) | [Previous: Developer Experience](07-developer-experience.md) | [Next: User Adoption Strategy →](09-user-adoption.md)

---

**Goal:** Strengthen the overall security posture of the Saga platform to match the increased risk profile that comes with instance consolidation, EHDS data exchange requirements, and expanding integration surfaces.

As Saga consolidates from many small instances into fewer shared deployments, the security equation changes fundamentally. A compromised distributed instance affected one organization; a compromised consolidated instance could expose data for many. Combined with EHDS mandating new external data exchange interfaces and the Silva/Ísland.is integrations expanding the attack surface, security must become a proactive, continuous investment — not an afterthought.

## 5.4.1 Authentication and Authorization

- **Centralized identity management.** As multiple organizations share a single deployment, the identity and access control layer must be robust. Evaluate whether the current authentication mechanism is fit for a multi-tenant consolidated environment. Consider adopting or strengthening an OpenID Connect / OAuth 2.0 based identity layer.
- **Role-based and organization-scoped access.** Every action in the system should be authorized both by the user's role and their organizational context. A clinician in Organization A must never be able to access data belonging to Organization B — and this must be enforced at the API level, not just the UI.
- **API authentication for integrations.** As the FHIR facade and Silva integration expose APIs externally, these must be secured with proper token-based authentication, scoped permissions, and rate limiting. No integration endpoint should be reachable without authentication.

## 5.4.2 Data Protection

- **Encryption at rest and in transit.** Verify that all data is encrypted at rest in Oracle (and PostgreSQL for any new modules) and that all communication between services, clients, and external systems uses TLS. This is a baseline requirement for EHDS compliance.
- **Tenant data isolation.** Beyond logical isolation in the application layer, consider database-level isolation strategies (schema-per-tenant, row-level security) to provide defense-in-depth against cross-tenant data leaks. Test isolation rigorously — a tenant isolation failure in a healthcare system is a critical incident.
- **Data classification.** Catalog the types of data Saga holds (PII, clinical data, billing data) and ensure that each category has appropriate access controls, retention policies, and audit requirements. EHDS secondary use provisions will require Saga to understand what data it holds and how it can be shared.
- **Secrets management.** Ensure API keys, database credentials, and integration tokens are stored in a secrets management solution (e.g., HashiCorp Vault, Kubernetes Secrets with encryption) rather than in configuration files or code.

## 5.4.3 Audit and Compliance

- **Comprehensive audit logging.** All access to patient data — reads and writes — should be logged with user identity, organizational context, timestamp, and the action performed. This is essential for EHDS compliance, incident investigation, and regulatory audits.
- **Immutable audit trail.** Audit logs must be tamper-resistant. Store them separately from application data, with restricted access and retention policies that meet healthcare regulatory requirements.
- **Access reviews.** Establish a process for periodic review of user access rights, particularly for privileged accounts and cross-organizational access in consolidated deployments.

## 5.4.4 Application Security

- **Dependency scanning.** Integrate automated vulnerability scanning for third-party dependencies (.NET NuGet packages, Angular npm packages) into the CI pipeline. Known vulnerabilities in dependencies are one of the most common attack vectors.
- **Static analysis and code review.** Adopt static application security testing (SAST) tools for .NET and Angular codebases. Security-sensitive code paths (authentication, authorization, data access, FHIR API endpoints) should require explicit security review.
- **Penetration testing.** As consolidated instances and new integration surfaces (FHIR, Silva, Ísland.is) go into production, schedule regular penetration testing — particularly for the multi-tenant boundaries and external-facing APIs.
- **Input validation and API hardening.** All external-facing APIs (FHIR facade, Silva integration, any Ísland.is-facing endpoints) must enforce strict input validation, rate limiting, and request size limits. The FHIR facade is a particularly sensitive surface given that it exposes clinical data.

## 5.4.5 Incident Response

- **Incident response plan.** Document a clear incident response process: who is notified, how is the incident contained, how are affected organizations informed, and what is the post-incident review process. With consolidated deployments, the incident response plan must account for multi-organization impact.
- **Security monitoring.** Extend the observability investment ([Section 5.3](07-developer-experience.md)) to include security-specific monitoring: failed authentication attempts, unusual data access patterns, API abuse, and privilege escalation attempts. Alerting should distinguish between operational issues and potential security incidents.

---

[← Back to Index](README.md) | [Previous: Developer Experience](07-developer-experience.md) | [Next: User Adoption Strategy →](09-user-adoption.md)
