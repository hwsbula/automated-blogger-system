# Implementation Plan

## Objective

Stand up the multi-client WordPress blogging system in a phased sequence that preserves auditability and safety from the first usable release.

## Build Sequence

### Milestone 0: Project and Policy Foundation

- Initialize the hybrid project workspace and core documents
- Finalize launch-vertical policy pack for web services
- Freeze phase 1 assumptions, route rationale, and approval thresholds

Exit criteria:
- architecture, technical spec, workflow map, failure modes, and SOPs are internally consistent

### Milestone 1: Tenant and Configuration Layer

- Build `clients`, `client_config_versions`, and `wordpress_connections`
- Implement `ClientConfig` validation and versioning
- Add intake completeness scoring and fallback-permission flags
- Add config-level auto-publish enablement and approval thresholds
- Add tenant taxonomy policy, blocked-term rules, and fallback category configuration

Exit criteria:
- operators can create and update a client config pack with audit history

### Milestone 2: Strategy and Briefing Layer

- Implement topic cluster records and editorial plan windows
- Build a planner that generates cluster priorities from services, locations, offers, exclusions, and seasonal weights
- Build brief generation with required claims, blocked claims, CTA goals, and internal-link targets
- Add duplicate-topic detection at planning and briefing time

Exit criteria:
- each client can produce a monthly plan and brief queue without duplicate cluster collisions

### Milestone 3: Drafting, Validation, and Gating

- Implement draft-package generation from briefs
- Extract claims from draft output and create claim records
- Implement source ledger attachment, freshness checks, usefulness scoring, and risk scoring
- Implement system and reviewer approval paths
- Add review queue creation for blocked or review-required drafts

Exit criteria:
- a draft can move from brief to explicit decision state with complete audit records

### Milestone 4: WordPress Publish Adapter

- Implement connection test flow for each client site
- Implement taxonomy catalog sync, alias mapping, and usage-aware reuse
- Implement category and tag auto-create only behind explicit tenant policy
- Implement create or update draft behavior
- Implement scheduled publish creation with WordPress `future` status and irregular publish-window assignment
- Add publish retry, rollback, and duplicate-prevention logic

Exit criteria:
- the system can publish a draft or scheduled post idempotently to a connected WordPress site
- existing WordPress categories and tags are reused before any new term is created

### Milestone 5: Monitoring and Operations

- Build queue views for review backlog, publish failures, stale-source alerts, and duplicate-topic incidents
- Implement incident logging and rollback procedures
- Finalize SOPs, QA checklist, and handoffs
- Perform artifact, system, and handoff QA

Exit criteria:
- an operator can onboard, review, publish, and maintain multiple clients without inventing missing steps

## Workstream Dependency Order

1. Master architecture and decision log
2. Data model and config schema
3. Planning and briefing
4. Drafting and validation
5. Approval queue and WordPress publishing
6. Monitoring and operational handoff
7. Independent QA pass

## Acceptance Criteria

- All nine pipeline stages exist with explicit inputs, outputs, and state transitions
- Validation is technically separate from generation
- WordPress is the only required direct integration in phase 1
- Auto-publish is policy-gated and disabled automatically for unsupported or high-risk content
- Post identity is idempotent across retries and duplicate detection
- Taxonomy reuse is deterministic and auditable before auto-create is attempted
- Every client has a versioned configuration pack and approval thresholds
- Every publish action leaves an internal audit trail

## Phase 1 Safe Scope

- Shared control plane
- Internal review console
- Web-services launch pack
- Topic strategy, briefing, drafting, validation, scoring, and WordPress publishing
- Internal observability and operator SOPs

## Deferred Scope

- Search Console and GA4 integrations
- CRM feedback loops
- richer client-facing collaboration tools
- plugin-specific SEO metadata sync as a required feature
- higher-risk vertical optimization beyond baseline policy packs
