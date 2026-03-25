# Decision Log

| ID | Date | Decision | Rationale | Impact |
| --- | --- | --- | --- | --- |
| D-001 | 2026-03-25 | Route the project as `R-HYBRID`. | Strategy, workflow, automation, integration, and operations are all first-order outcomes. | Multiple workstreams and cross-phase deliverables are required. |
| D-002 | 2026-03-25 | Use multiple workstreams with independent QA. | Claim safety and publishing behavior need clear ownership and review separation. | The project follows the repo threading standard for hybrid work. |
| D-003 | 2026-03-25 | Use a shared multi-tenant Python control plane with per-client configuration packs. | This fits a productized service model while keeping operations manageable. | Clients share core infrastructure but retain isolated configs, credentials, and audit records. |
| D-004 | 2026-03-25 | Keep WordPress as the only required direct application integration in phase 1. | The user explicitly constrained phase 1 to WordPress-only direct integration. | Performance feedback loops must rely on internal or manual inputs until later phases. |
| D-005 | 2026-03-25 | Separate generation from validation and approval. | Raw model output is not trusted and must not self-certify. | The system includes discrete validation, scoring, and gating stages. |
| D-006 | 2026-03-25 | Store source ledgers, claim records, and publish history in Postgres rather than relying on WordPress meta. | Plugin-agnostic design requires an internal system of record. | WordPress remains the publish surface while internal data stays auditable and portable. |
| D-007 | 2026-03-25 | Use WordPress Application Passwords as the default auth model. | It is broadly supported and avoids heavier custom authentication for phase 1. | Each client site needs a dedicated service account and HTTPS enforcement. |
| D-008 | 2026-03-25 | Use a database-backed job model instead of an external scheduler or queue dependency. | The system must avoid third-party scheduling tools and minimize dependencies. | Workers poll Postgres job tables and use WordPress native scheduled publish dates. |
| D-009 | 2026-03-25 | Make web services the launch vertical and defer solar to a later phase. | Solar has materially higher proof and compliance risk. | Vertical policy packs ship in a controlled order. |
| D-010 | 2026-03-25 | Require hard review or block states for proof-sensitive claims regardless of score. | Numeric scores alone are not sufficient for high-risk content. | Guarantees, compliance statements, comparison claims, and stale sources bypass auto-publish eligibility. |
