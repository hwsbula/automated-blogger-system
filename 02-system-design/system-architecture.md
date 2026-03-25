# System Architecture

## System Objective

Build a reusable multi-client content system that converts client inputs into validated WordPress drafts or scheduled posts while maintaining source traceability, claim safety, usefulness standards, and operator control over risk.

## Design Principles

- Favor production-safe behavior over throughput
- Separate generation from validation, scoring, and approval
- Treat WordPress as a publishing destination, not the full compliance record
- Use per-client configuration packs instead of bespoke logic branches
- Make every publishable artifact auditable
- Prefer fewer external dependencies when the tradeoff does not reduce control

## Tenancy Model

Phase 1 uses a shared multi-tenant control plane with logical tenant isolation.

- Each client has its own configuration pack, WordPress credential record, policy overrides, review history, and publish-state history.
- Shared services include the internal admin UI, API, worker processes, database, logging, and monitoring.
- Tenant boundaries are enforced at the application and data-query level, with unique tenant-scoped identifiers and row-level ownership rules in the service layer.

## Core Components

1. Intake and configuration service
   - Stores `ClientConfig`
   - Validates required fields, fallback permissions, and tenant taxonomy policy
   - Versions config changes for auditability
2. Strategy and planning engine
   - Builds topic clusters, cluster priorities, and editorial windows
   - Produces briefs from client goals, services, locations, and approved research
3. Draft generation pipeline
   - Converts briefs into structured draft packages
   - Inserts CTAs, internal-link suggestions, excerpt options, and a reuse-first taxonomy plan
4. Validation and evidence service
   - Maintains source ledgers, claim records, freshness checks, and support status
   - Rejects unsupported or prohibited claims before publish actions
5. Usefulness and risk scorer
   - Scores posts for search intent fit, local value, offer alignment, originality, and risk
   - Assigns draft, review, scheduled, auto-publish, or block states
6. Review queue and approval console
   - Presents blocked and review-required items to operators
   - Records reviewer decisions, rationale, overrides, and taxonomy exceptions
7. WordPress publishing adapter
   - Creates or updates drafts and scheduled posts through the REST API
   - Reuses existing categories and tags before any term creation
   - Syncs categories, tags, slug, excerpt, and post status
8. Refresh and optimization loop
   - Reviews aging content, outdated sources, and underutilized clusters
   - Creates refresh jobs without requiring external analytics in phase 1
9. Observability and control layer
   - Tracks queue health, publish failures, validation failures, stale sources, and duplicate-topic collisions

## Control Plane Stack

- Python 3.12 service
- FastAPI for internal API and ops-facing UI endpoints
- Server-rendered templates for the review console in phase 1
- Postgres for primary storage, audit records, and job queues
- SQLAlchemy and Alembic for persistence and schema migration
- Separate web and worker processes running against the same database

This stack avoids third-party scheduling tools and keeps the service auditable and deployment-light.

## Data Model Overview

### Tenant and Configuration Records

- `clients`
- `client_config_versions`
- `wordpress_connections`
- `vertical_policy_packs`

### Planning and Drafting Records

- `topic_clusters`
- `editorial_plan_windows`
- `post_briefs`
- `draft_packages`

### Validation and Audit Records

- `source_documents`
- `source_ledger_entries`
- `claim_records`
- `validation_runs`
- `usefulness_scores`
- `risk_scores`
- `approval_decisions`

### Publishing and Operations Records

- `publish_jobs`
- `wordpress_post_links`
- `publish_state_history`
- `review_tasks`
- `refresh_candidates`
- `incident_log`

## Pipeline Stages

### 1. Client Intake and Configuration

Inputs:
- client name
- business type
- vertical
- services
- locations
- offers
- voice rules
- prohibited and proof-backed claims
- CTA rules
- cadence settings
- taxonomy policy and blocked-term list
- internal-link targets
- content exclusions
- WordPress connection info

Outputs:
- validated `ClientConfig`
- config version
- intake completeness score
- fallback permissions

Rules:
- incomplete intake is allowed only when fallback policy stays within low-risk informational context
- prohibited inference categories cannot be bypassed by score or operator convenience

### 2. Topic Strategy and Cluster Planning

Outputs:
- prioritized topic clusters
- monthly targets
- seasonal windows
- cluster-level publish spacing

Strategy rules:
- prioritize service-plus-location, buyer-education, objection-handling, and local authority topics for web services
- avoid producing multiple near-duplicate topics within the same cluster window
- keep derivative potential for email nurture as metadata, not as a required integration

### 3. Post Brief Generation

Each brief includes:
- target query or intent
- audience and funnel stage
- service and location angle
- outline
- primary category hint
- secondary category hint if justified
- candidate tags or a `no_useful_tags` reason
- approved claims
- prohibited claims reminder
- required sources and freshness requirements
- CTA goal
- internal-link targets

### 4. Draft Generation

The generator may use only:
- brief instructions
- approved client facts
- approved public-source context
- allowed inference categories

The generator may not assert:
- price numbers
- guarantee language
- savings amounts
- certifications
- warranties
- legal or compliance status
- performance claims without proof

### 5. Factual Validation and Claim Checking

Validation is a separate service stage that:
- extracts explicit and implicit claims
- maps each claim to one or more source ledger entries
- assigns support status: `supported`, `partially_supported`, `unsupported`, `prohibited_without_confirmation`
- applies freshness rules by claim type

### 6. Usefulness and Risk Scoring

Usefulness rubric, maximum 35:
- search intent fit: 0-5
- local relevance: 0-5
- offer alignment: 0-5
- originality: 0-5
- internal-link opportunity: 0-5
- CTA relevance: 0-5
- practical usefulness: 0-5

Thresholds:
- below 24: reject or re-brief
- 24-27: draft eligible, not auto-publish eligible
- 28-35: quality threshold met for scheduled or auto-publish consideration

Risk rubric, maximum 100:
- vertical baseline risk: 0-20
- topic type risk: 0-20
- claim sensitivity: 0-25
- source quality weakness: 0-15
- source freshness weakness: 0-10
- brand and reputational sensitivity: 0-10

State rules:
- `0-24`: low-risk if no hard override exists
- `25-59`: requires human review
- `60+`: blocked until re-scoped or proven

Hard overrides:
- proof-sensitive claims
- comparison claims naming competitors
- guarantee statements
- compliance or legal language
- stale required sources
- unsupported factual assertions

### 7. Approval Gating

Allowed states:
- `auto_publish`
- `save_scheduled`
- `draft_only`
- `require_review`
- `blocked`

Gate logic:
- `auto_publish` only if usefulness is at least 28, risk is at most 24, all claims are supported, no hard override exists, and the client policy pack enables auto-publish for the topic type
- `save_scheduled` only after human approval for items eligible to be scheduled but not auto-published
- `draft_only` for content that is useful but not ready to schedule
- `require_review` when any manual check is required
- `blocked` when prohibited or unsupported claims remain

### 8. WordPress Draft or Scheduled Publish Creation

The adapter:
- refreshes the tenant taxonomy cache
- resolves category and tag IDs through slug, normalized name, alias, and usage-aware reuse
- creates new terms only when no reusable term exists and tenant policy allows auto-create
- creates or updates the post
- assigns slug, excerpt, content, status, schedule, categories, and tags
- requires at least one non-blocked category before create or update
- preserves the internal-to-WordPress mapping
- records every publish transition in `publish_state_history`

### 9. Refresh, Pruning, and Optimization Loop

Refresh triggers:
- stale source ledger entries
- aging content with outdated platform guidance
- cluster gaps
- manual editorial feedback
- publish failures that resulted in long-lived drafts

Phase 1 decisions are based on internal records and manual feedback rather than external analytics.

## Source Policy

### Tier 1

Client-provided and explicitly approved proof:
- service documents
- approved website copy
- pricing or policy documents
- case studies
- signed-off claim library

### Tier 2

Primary public authorities:
- official platform documentation
- government or municipal resources
- standards bodies
- official vendor documentation when relevant

### Tier 3

Reputable secondary sources:
- established industry publishers
- widely cited research reports
- trade associations without direct sales conflict

### Tier 4

Contextual ideation only:
- competitor sites
- community discussions
- generic blog posts without primary sourcing

Tier 4 may inspire topics but cannot support publishable claims on its own.

## Fallback Policy for Weak Client Inputs

Allowed:
- infer low-risk informational context from the client website
- infer low-risk informational context from approved public research
- generalize non-claiming educational guidance for the launch vertical

Not allowed without confirmation:
- pricing
- guarantees
- savings
- certifications
- warranties
- legal or compliance statements
- hard performance claims

## Vertical and Topic Risk Framework

| Vertical | Baseline Risk | Notes |
| --- | --- | --- |
| Web services | low | Launch vertical. Most educational and process topics are manageable with source discipline. |
| General contractors | medium | More proof-sensitive service, timeline, and permit claims. |
| Ebike rental shops | medium | Local policy, safety, and liability topics increase review needs. |
| Ecommerce | medium | Shipping, returns, fulfillment, and comparative claims raise risk. |
| Solar contractors | high | Incentives, savings, compliance, and hardware-performance claims require tighter controls. |

| Topic Type | Default Risk | Notes |
| --- | --- | --- |
| General educational explainer | low | Eligible for auto-publish when all thresholds pass. |
| Service process guide | low to medium | Review required if it references local rules, timelines, or outcomes. |
| Comparison or alternatives post | medium to high | Human review required by default. |
| Pricing or cost post | high | Proof-backed only; otherwise blocked. |
| Incentives, compliance, or legal guidance | high | Human review and fresh authoritative sources required. |
| Results or case-study post | high | Explicit proof required. |

## Cadence Design

- Define a stable monthly target per client, not a rigid weekly slot
- Generate publish windows rather than fixed dates
- Randomize within policy-approved windows while respecting:
  - minimum gap between publishes
  - maximum idle gap
  - cluster spacing
  - seasonality
  - business-day preferences
- Avoid repeating the same weekday and time pattern across a month
- Prefer bursts around seasonal or service-specific relevance when policy allows

## Monitoring and Operational Controls

Track:
- publish failure rate
- validation failure rate
- unsupported-claim blocks
- duplicate-topic collision rate
- stale-source count
- review queue age and backlog
- manual rollback and unpublish incidents

Control actions:
- pause auto-publish per client or per vertical
- downgrade a policy pack from auto-publish to review-only
- requeue transient publish jobs
- force content into draft-only when upstream quality degrades

## Phase Boundaries

### Safe in Phase 1

- Internal intake and config management
- Topic planning, briefing, drafting, validation, and approval
- WordPress draft and scheduled publish creation
- Internal review queue and audit history
- Refresh and pruning based on age, source freshness, and manual feedback

### Deferred to Later Phases

- Search Console-driven content prioritization
- GA4-driven refresh automation
- CRM or lead-attribution feedback loops
- Client-facing portals
- SEO-plugin-specific adapters beyond the optional integration layer
