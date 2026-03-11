# 9. Risks and Mitigations

[← Back to Index](README.md) | [Previous: Guiding Principles](11-guiding-principles.md)

---

| Risk | Impact | Mitigation |
|------|--------|------------|
| FHIR implementation complexity underestimated | Delays in integration delivery | Start with a minimal facade (2–3 resources), validate with a real integration partner early |
| Legacy module migration takes longer than expected | Prolonged maintenance of dual systems | Strict scoping of migration units; accept that some legacy will persist beyond this horizon |
| Team capacity stretched across modernization and feature work | Neither gets done well | Dedicate a portion of each sprint to infrastructure/modernization work (e.g., 20–30%) rather than treating it as a separate project |
| EHDS compliance timeline is aggressive for current interoperability maturity | Non-compliance, loss of market position | Start with gap assessment immediately; align FHIR work to EHDS priority data categories; engage early with Icelandic digital health authority |
| Oracle licensing costs limit scaling and environments | Expensive test/staging environments, vendor lock-in | Abstract data access in new code, evaluate PostgreSQL for greenfield modules, catalog Oracle-specific dependencies |
| Consolidated instances increase security risk and blast radius | Multi-organization data breach | Security hardening program: multi-tenant access controls, audit logging, penetration testing, dependency scanning, incident response planning |
| Instance consolidation causes data or configuration issues | Disruption for organizations being migrated | Build repeatable migration tooling, run consolidations incrementally, validate data isolation rigorously |
| Nordic expansion requirements are unclear | Architectural rework later | Engage early with potential Nordic partners/customers to understand requirements; design for extensibility |
| User resistance to SagaPlus modules | Low adoption undermines migration ROI | Feature flags, gradual rollout, telemetry-driven iteration, and tight feedback loops |
| Shell technology choice doesn't meet requirements | Rework or technology switch mid-stream | Build proof-of-concept early, test against key requirements (Angular hosting, IPC, cross-platform), make decision before committing to production development |
| Process bridge IPC adds complexity and latency | Jarring user experience between legacy and modern modules | Start with minimal context sharing (patient, session), iterate based on user feedback, set performance budgets for context switches |
| Journal aggregation performance | Slow Journal kills adoption | Design with caching, lazy loading, and progressive rendering from day one; set performance budgets |
| Silva ↔ Saga integration complexity | Scheduling conflicts, data inconsistency | Define clear API contracts early, implement conflict resolution, test with realistic concurrent usage scenarios |
| Government trust erodes despite technical efforts | Political pressure to replace Saga | Engage proactively, deliver on transparency commitments early, treat government relationship as a first-class priority |

---

*This is a living document. It should be revisited quarterly and updated as the team learns from implementation experience and evolving business priorities.*

---

[← Back to Index](README.md) | [Previous: Guiding Principles](11-guiding-principles.md)
