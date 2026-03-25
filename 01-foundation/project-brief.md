# Project Brief

## Objective

Design a reusable multi-client blogging system for a web design business that produces strategy-driven blog plans, generates drafts, validates claims and usefulness independently from generation, and publishes approved content to WordPress as drafts or scheduled posts through the WordPress REST API.

The system must support low-risk auto-publish, mandatory human review for proof-sensitive content, natural publishing cadence, and a per-client configuration model that scales across verticals without turning into a bulk content spinner.

## Route ID

R-HYBRID

## Project Path

projects/active/multi-client-wordpress-blogging-system/

## Recommended Execution Model

- Multiple workstreams
- This project spans strategy, workflow design, validation policy, WordPress integration, operating procedures, and QA, which matches the repository standard for `R-HYBRID`.

## Workstreams

- Master systems architecture
- Client configuration and intake design
- Content strategy and editorial planning engine
- Content generation and enrichment pipeline
- Validation and risk-gating pipeline
- WordPress publishing and scheduling integration
- QA, monitoring, and operational handoff
- QA reviewer

## Deliverables

- `01-foundation/project-brief.md`
- `01-foundation/assumptions-log.md`
- `01-foundation/decision-log.md`
- `01-foundation/threading-decision.md`
- `02-system-design/system-architecture.md`
- `02-system-design/technical-spec.md`
- `02-system-design/implementation-plan.md`
- `02-system-design/integrations.md`
- `03-workflows/workflow-map.md`
- `03-workflows/failure-modes.md`
- `04-operations/sops.md`
- `04-operations/qa-checklist.md`
- `04-operations/handoffs.md`
- `05-delivery/final-deliverables.md`

## Business Context

- Productized multi-client system, not a bespoke client build
- Launch vertical: web services
- Later verticals: general contractors, ebike rental shops, solar contractors, ecommerce, and adjacent service businesses
- Business goals: SEO traffic, local lead generation, topical authority, sales enablement, and email-nurture derivatives

## Success Criteria

- An operator can onboard a new WordPress client with a deterministic configuration pack and no custom redesign of the system
- Every post flows through separate generation, validation, scoring, approval, and publish stages with audit history
- Low-risk informational content can auto-schedule safely when thresholds pass
- Proof-sensitive or unsupported claims are blocked from auto-publish and routed into review or block states
- Publishing cadence looks editorial rather than mechanical
- Phase 1 works with WordPress as the only direct application integration

## Scope

### In Scope

- Shared multi-tenant control plane
- Per-client configuration schema and intake rules
- Topic planning, post briefing, drafting, validation, risk scoring, and gating
- WordPress draft and scheduled publish creation
- Audit metadata, review history, and publish-state history
- QA, monitoring, SOPs, and handoffs
- Refresh, pruning, and optimization workflow that can run without external analytics integrations

### Out of Scope

- Search Console, GA4, CRM, and ESP integrations in phase 1
- Auto-inference of pricing, guarantees, savings, certifications, warranties, legal statements, or performance claims without proof
- Plugin-specific WordPress SEO integration as a hard dependency
- Fully automated client-facing portals in phase 1
