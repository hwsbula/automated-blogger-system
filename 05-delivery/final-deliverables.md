# Final Deliverables

## 1. Objective

Design a reusable, auditable, multi-client blogging system for a web design business that creates strategy-driven blog plans, drafts posts, validates claims and usefulness, and publishes safe content to WordPress as drafts or scheduled posts while keeping high-risk topics under human review.

## 2. Recommended execution model

- `R-HYBRID` multiple-workstream execution
- Reason: the system spans strategy, workflow automation, validation policy, WordPress integration, operations, and QA

## 3. Route selection and rationale

- Route: `R-HYBRID`
- Rationale:
  - business architecture and technical execution are equally important
  - reusable operations are a first-order deliverable
  - the system requires coordinated outputs across foundation, system design, workflows, operations, and delivery
  - QA must stay independent from production generation because claim safety is release-critical

## 4. Assumptions log

- `A-001` Web services is the launch vertical.
- `A-002` Human review is internal in phase 1.
- `A-003` WordPress auth uses HTTPS plus Application Passwords per client site.
- `A-004` Postgres is the authoritative audit and workflow record.
- `A-005` Weak intake may use low-risk inference from the client site and approved public research only.
- `A-006` Pricing, guarantees, savings, certifications, warranties, legal or compliance statements, and hard performance claims cannot be inferred without proof.
- `A-007` Phase 1 optimization runs without GSC, GA4, or CRM integrations.
- `A-008` Hosting is vendor-agnostic if it supports web, workers, and Postgres.
- `A-009` The review console can be server-rendered in the Python service.
- `A-010` Audit metadata remains internal by default rather than depending on WordPress meta.

## 5. System architecture

Core system:
- shared multi-tenant Python control plane
- FastAPI API and internal review console
- Postgres for tenant data, source ledgers, claims, scores, approvals, publish history, incidents, and job queues
- worker processes for planning, briefing, drafting, validation, scoring, publishing, and refresh scans
- WordPress REST adapter as the only required direct application integration

Fixed pipeline:
1. client intake and configuration
2. topic strategy and cluster planning
3. post brief generation
4. draft generation
5. factual validation and claim checking
6. usefulness scoring and risk scoring
7. approval gating
8. WordPress draft or scheduled publish creation
9. refresh, pruning, and optimization loop

Decision states:
- `auto_publish`
- `save_scheduled`
- `draft_only`
- `require_review`
- `blocked`

## 6. Workstreams and ownership

- Master systems architecture: route rationale, architecture, phase boundaries, and merge logic
- Client configuration and intake: tenant schema, config versioning, fallback permissions, and onboarding rules
- Content strategy and editorial planning: topic clusters, briefs, cadence, refresh logic, and phased vertical rollout
- Content generation and enrichment: draft structure, CTA insertion, internal linking, and excerpt guidance
- Validation and risk gating: source policy, claim support, freshness checks, usefulness rubric, and risk scoring
- WordPress publishing and scheduling: authentication, post create or update, taxonomy mapping, schedule assignment, retries, and idempotency
- QA, monitoring, and operational handoff: SOPs, incidents, rollback, backlog management, and handoff packet
- QA reviewer: artifact, system, and handoff QA

## 7. Client configuration model

Minimum schema:
- `client_id`
- `business_type`
- `vertical`
- `services[]`
- `locations[]`
- `offers[]`
- `voice_rules`
- `prohibited_claims[]`
- `proof_backed_claims[]`
- `internal_link_targets[]`
- `cta_rules`
- `approval_thresholds`
- `cadence_settings`
- `content_exclusions[]`
- `source_policy_overrides`
- `wordpress_connection`

Fallback policy:
- allowed: low-risk informational inference from the client website and approved public research
- not allowed without confirmation: pricing, guarantees, savings claims, certifications, warranties, legal or compliance statements, hard performance claims

## 8. Content strategy and validation framework

Strategy model:
- build clusters from services, locations, offers, search intent, and local lead goals
- prefer service-plus-location, educational, objection-handling, and topical-authority content for the launch vertical
- keep email-derivative potential as metadata for later use

Source policy tiers:
- Tier 1: client-approved proof and assets
- Tier 2: primary public authorities
- Tier 3: reputable secondary sources
- Tier 4: contextual ideation only

Freshness rules:
- evergreen informational: 24 months
- platform or process guidance: 6 months
- local, legal, incentive, or compliance topics: 90 days

Usefulness rubric:
- 0-35 across search intent fit, local relevance, offer alignment, originality, internal-link opportunity, CTA relevance, and practical usefulness
- minimum 24 to remain in play
- minimum 28 for auto-publish eligibility

Risk rubric:
- 0-24 low-risk if no hard override exists
- 25-59 review required
- 60+ blocked
- proof-sensitive claims, comparison claims, guarantee language, compliance language, stale sources, and unsupported claims bypass numeric thresholds and force review or block

## 9. WordPress integration design

Authentication:
- HTTPS plus Application Passwords with a dedicated service account per client

Core REST behavior:
- create or update posts through `/wp-json/wp/v2/posts`
- resolve categories and tags through reuse-first REST lookups and cached alias rules
- create new categories or tags only when no reusable live term exists and tenant policy allows it
- assign `title`, `content`, `slug`, `excerpt`, `categories`, `tags`, `author`, and optional `featured_media`
- use `status=draft` for draft workflows
- use `status=future` plus `date_gmt` for scheduled publish

Controls:
- deterministic slug generation
- unique content identity on `client_id + canonical_topic_key + content_cycle`
- require at least one non-blocked category before create or update
- retry only for transient failures
- log every publish attempt and state change internally
- keep plugin-specific metadata in an optional adapter layer only

## 10. Failure modes and risk controls

Primary risks:
- unsupported or prohibited claims
- stale authoritative sources
- invalid WordPress credentials
- duplicate-topic or slug collisions
- missing taxonomy mappings
- review backlog
- publish retries causing duplication
- factual disputes after scheduling or publish

Controls:
- separate validation stage before publish queue entry
- freshness checks during validation and before scheduled publish
- duplicate detection during planning and before publish
- policy-driven block states
- rollback and unpublish procedures with incident logging
- ability to pause auto-publish globally, by client, or by vertical

## 11. QA framework

Artifact QA:
- verify objective match, route consistency, thresholds, schema completeness, and file placement

System QA:
- verify stage ordering, failure coverage, dependency clarity, monitoring coverage, and phase boundaries

Handoff QA:
- verify ownership, next actions, open decisions, and review-first files

Outcome states:
- pass
- revise
- block

## 12. Phased rollout plan

Vertical rollout:
1. web services
2. general contractors
3. ebike rental shops
4. ecommerce
5. solar

Rationale:
- increase proof burden gradually
- keep solar last because incentives, savings, compliance, and hardware-performance claims create materially higher risk

Phase 1 safe scope:
- intake and configuration
- topic planning and briefs
- draft generation
- validation and risk gating
- internal review queue
- WordPress draft and scheduled publish
- audit history, incidents, and manual refresh loop

Deferred:
- Search Console integrations
- GA4 integrations
- CRM feedback loops
- client-facing portal
- required SEO-plugin adapters
- advanced solar policy packs

## 13. Deterministic file structure

```text
projects/active/multi-client-wordpress-blogging-system/
  01-foundation/
    project-brief.md
    assumptions-log.md
    decision-log.md
    threading-decision.md
  02-system-design/
    system-architecture.md
    technical-spec.md
    implementation-plan.md
    integrations.md
  03-workflows/
    workflow-map.md
    failure-modes.md
  04-operations/
    sops.md
    qa-checklist.md
    handoffs.md
  05-delivery/
    final-deliverables.md
```

## 14. Implementation plan

Milestones:
1. policy and project foundation
2. tenant and configuration layer
3. strategy and briefing layer
4. drafting, validation, and gating
5. WordPress publish adapter
6. monitoring and operations

Completion standard:
- the operator can onboard a client, generate safe topics, create drafts, validate them, send them through approval, and write drafts or scheduled posts to WordPress without inventing missing logic

## 15. Open decisions and next actions

Open decisions:
- deployment provider
- whether any clients need stricter tenant isolation than logical partitioning
- which optional SEO meta adapters, if any, are worth implementing after phase 1

Next actions:
1. convert the technical spec into implementation tickets by milestone
2. build the tenant, config, and job-queue tables first
3. implement validation and gating before draft-to-WordPress automation
4. build the internal review queue before enabling any auto-publish paths
5. launch with web services only and keep auto-publish scoped to low-risk informational topics
