# Assumptions Log

| ID | Assumption | Why It Was Made | Risk | Confirm Later |
| --- | --- | --- | --- | --- |
| A-001 | Web services is the launch vertical and the baseline policy pack for phase 1. | The user stated web services is the first launch vertical. | low | no |
| A-002 | Human review happens in an internal operator queue, not a client-facing portal. | Internal review is sufficient for phase 1 and keeps dependencies low. | low | yes |
| A-003 | WordPress authentication uses HTTPS and Application Passwords with one dedicated service account per client site. | This is the safest default that works broadly with the WordPress REST API in phase 1. | low | yes |
| A-004 | Postgres is the authoritative record for audit data, validation state, approval history, and publish-state history. | WordPress should remain the publishing target rather than the compliance system of record. | low | no |
| A-005 | Weak client intake can be supplemented by approved public research and the client website only for low-risk informational context. | The user allowed public research and site inference for low-risk context. | low | no |
| A-006 | Pricing, guarantees, savings, certifications, warranties, legal or compliance statements, and hard performance claims cannot be inferred without explicit proof. | The user listed these as prohibited inference categories. | low | no |
| A-007 | Phase 1 refresh and optimization decisions rely on content age, manual review notes, and WordPress state rather than external analytics. | External analytics integrations are explicitly deferred. | low | no |
| A-008 | Hosting is non-blocking if the environment can run one web service, worker processes, and managed Postgres. | The plan does not require vendor-specific infrastructure. | low | yes |
| A-009 | The internal review console can be implemented as server-rendered pages on the Python service rather than a separate SPA. | This minimizes dependencies and is sufficient for an ops-facing tool. | low | yes |
| A-010 | Phase 1 stores authoritative audit metadata internally and syncs only publishable post fields to WordPress by default. | Plugin-agnostic design requires the internal system to hold more metadata than WordPress can reliably store without custom registration. | low | no |
