# 8. Guiding Principles

[← Back to Index](README.md) | [Previous: Prioritized Roadmap](10-roadmap.md) | [Next: Risks and Mitigations →](12-risks-mitigations.md)

---

1. **Do fewer things well.** We have a vast system to support with a small team. Rather than spreading thin across many purpose-built solutions, invest deeply in flexible building blocks — Sheets/Meðvera, the shell module contract, well-designed APIs — that can absorb a wide range of requirements. Accept that modular and configurable sometimes means less polished than bespoke, and that's the right trade-off for sustainability.
2. **Reuse, integrate, then build.** When a new feature request arrives, ask in order: can this be a new sheet in Meðvera? Can a third-party product handle it through our integration APIs? Only if neither works should we build a new module. This keeps the codebase lean and the team focused.
3. **Incremental over big-bang.** Every change should be deliverable in small, safe steps. No multi-month migrations without intermediate value.
4. **Pragmatic boundaries.** Define clear domain boundaries, but don't over-engineer into microservices prematurely. A modular monolith is fine for a team of our size.
5. **Standards where they matter.** Adopt FHIR for external interoperability and EHDS compliance, but don't force internal systems into FHIR's data model.
6. **Automate the boring stuff.** Invest in CI/CD, test infrastructure, and environment provisioning so the team spends time on product value, not manual processes.
7. **Measure before migrating.** Use telemetry and user feedback to prioritize which legacy modules to replace, rather than guessing.
8. **Document decisions, not just code.** ADRs and playbooks compound in value as the team evolves.
9. **Compliance as enabler, not burden.** Frame EHDS and interoperability work as investments that open markets and integrations, not just regulatory checkboxes.
10. **Earn trust through transparency.** Data ownership, source availability, and standards-based interoperability are not concessions — they are competitive advantages that build lasting government and institutional trust.

---

[← Back to Index](README.md) | [Previous: Prioritized Roadmap](10-roadmap.md) | [Next: Risks and Mitigations →](12-risks-mitigations.md)
