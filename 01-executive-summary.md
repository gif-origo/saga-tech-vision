# 1. Executive Summary

[← Back to Index](README.md) | [Next: Where We Are Today →](02-current-state.md)

---

Saga is a multi-purpose healthcare platform serving the Icelandic health system, with ambitions to expand into the Nordic market. The system has evolved over many years, resulting in a layered architecture spanning legacy Delphi components, a WPF desktop client, and modern Angular-based modules (SagaPlus) backed by .NET services.

This document outlines a 1–2 year technical vision organized around **four product pillars** — Registration, Journal, Billing, and Scheduling — and supported by a **data sovereignty and transparency strategy** to address government concerns, and four **technical pillars**: modernizing the architecture (including a new multi-platform shell), adopting healthcare interoperability standards to meet EHDS regulatory requirements, improving developer experience, and hardening security.

A core strategic principle runs through this vision: **do fewer things well.** Saga is a vast system with broad responsibilities, and the team is not large enough to build and maintain purpose-built solutions for every feature request. Instead, we favor modular, flexible building blocks — particularly the Sheets platform and its configurable form engine — that can absorb new requirements through configuration rather than new code. When a feature request arrives, the first question should be: can this be solved with a new sheet, an adjusted configuration, or a third-party integration — before we consider building a new module. This philosophy extends to our integration strategy: we will actively bring in third-party solutions for specialized capabilities, which makes a strong developer portal and standardized integration APIs essential infrastructure.

Together, these investments will position Saga for sustainable growth — both as a product and as a codebase the team can confidently evolve.

---

[← Back to Index](README.md) | [Next: Where We Are Today →](02-current-state.md)
