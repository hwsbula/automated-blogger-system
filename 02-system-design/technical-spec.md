# Technical Spec

## Objective

Define the phase 1 implementation blueprint for a multi-client blogging control plane that manages configuration, planning, drafting, validation, approval, and WordPress publishing with auditable state transitions.

## Recommended Stack

- Python 3.12
- FastAPI for internal API and server-rendered ops console
- Pydantic for request and config validation
- SQLAlchemy plus Alembic for persistence and migrations
- Postgres for relational data, JSON fields, audit logs, and job queues
- Separate worker processes that poll database job tables and use advisory locks

## Service Boundaries

### 1. Control Plane API

Responsibilities:
- CRUD for clients and configuration versions
- queue orchestration endpoints
- reviewer actions
- publish retry and rollback actions

Suggested endpoints:
- `POST /clients`
- `PUT /clients/{client_id}/config`
- `GET /clients/{client_id}/config`
- `POST /clients/{client_id}/plans/generate`
- `POST /briefs/{brief_id}/draft`
- `POST /drafts/{draft_id}/validate`
- `POST /drafts/{draft_id}/score`
- `POST /drafts/{draft_id}/approve`
- `POST /drafts/{draft_id}/publish`
- `POST /wordpress/posts/{internal_post_id}/retry`
- `POST /wordpress/posts/{internal_post_id}/rollback`

### 2. Review Console

Responsibilities:
- intake completeness review
- blocked-content review
- manual approval for medium and high-risk content
- audit trail inspection

Console screens:
- clients
- config versions
- editorial queue
- review queue
- publish job queue
- incidents

### 3. Worker Layer

Worker job types:
- `plan_generation`
- `brief_generation`
- `draft_generation`
- `validation_run`
- `risk_scoring`
- `approval_transition`
- `wordpress_publish`
- `refresh_scan`

Execution model:
- workers claim queued rows with advisory locking
- jobs are idempotent and retryable
- each job writes structured status, attempt count, started timestamp, finished timestamp, and error class

### 4. WordPress Adapter

Responsibilities:
- credential test
- taxonomy resolution
- post create or update
- scheduled publish date assignment
- duplicate detection support

## Core Schemas

### ClientConfig

```json
{
  "client_id": "cli_123",
  "business_type": "web design agency",
  "vertical": "web_services",
  "services": ["website design", "website redesign", "local seo"],
  "locations": ["Honolulu, HI", "Oahu, HI"],
  "offers": ["free website audit", "discovery call"],
  "voice_rules": {
    "tone": ["clear", "direct", "non-hype"],
    "avoid": ["guarantee language", "aggressive urgency"],
    "reading_level": "business owner"
  },
  "prohibited_claims": ["guaranteed ranking gains", "specific ROI numbers without proof"],
  "proof_backed_claims": [
    {
      "claim": "10+ years designing service business websites",
      "proof_reference": "client-approved brand document"
    }
  ],
  "internal_link_targets": [
    {"url": "/services/web-design/", "anchor_intents": ["web design services", "website design help"]}
  ],
  "cta_rules": {
    "primary_cta": "Book a discovery call",
    "secondary_cta": "Request a website audit",
    "allowed_cta_types": ["consultation", "audit"]
  },
  "approval_thresholds": {
    "auto_publish_max_risk": 24,
    "auto_publish_min_usefulness": 28,
    "draft_min_usefulness": 24
  },
  "cadence_settings": {
    "monthly_target": 6,
    "min_gap_days": 3,
    "max_gap_days": 9,
    "preferred_publish_windows": ["Tue-Thu 08:00-15:00"],
    "avoid_repeating_exact_pattern": true
  },
  "content_exclusions": ["lawsuit topics", "salary topics"],
  "source_policy_overrides": {
    "require_client_approval_for_case_studies": true
  },
  "taxonomy_policy": {
    "required_category_count": 1,
    "max_category_count": 2,
    "min_tag_count": 0,
    "max_tag_count": 5,
    "allow_zero_tags_with_reason": true,
    "category_auto_create": false,
    "tag_auto_create": false,
    "reuse_precedence": ["slug_exact", "name_casefold", "approved_alias", "highest_usage"],
    "blocked_term_slugs": ["uncategorized"],
    "fallback_category_slug": "insights"
  },
  "wordpress_connection": {
    "site_url": "https://example.com",
    "username": "blog-automation",
    "application_password_secret_ref": "wp_cli_123"
  }
}
```

### TopicCluster

```json
{
  "cluster_id": "clu_001",
  "client_id": "cli_123",
  "theme": "website redesign planning",
  "business_goal": "lead_generation",
  "target_services": ["website redesign"],
  "target_locations": ["Honolulu, HI"],
  "priority_score": 82,
  "seasonality_weight": 0.4,
  "monthly_post_target": 2
}
```

### PostBrief

```json
{
  "brief_id": "brf_001",
  "client_id": "cli_123",
  "canonical_topic_key": "web-design-cost-honolulu-guide",
  "intent_type": "commercial_informational",
  "audience": "small business owner evaluating a redesign",
  "outline": ["intro", "cost drivers", "scope choices", "when to talk to an agency"],
  "primary_category_hint": "website redesign",
  "secondary_category_hints": ["small business websites"],
  "tag_hints": ["website redesign", "website planning", "honolulu small business websites"],
  "no_useful_tags_reason": null,
  "allowed_claims": ["general process guidance"],
  "blocked_claims": ["specific price ranges unless client-approved"],
  "required_source_types": ["tier1_or_tier2"],
  "cta_goal": "book discovery call"
}
```

### SourceLedgerEntry

```json
{
  "source_id": "src_001",
  "client_id": "cli_123",
  "tier": 2,
  "source_type": "official_platform_docs",
  "title": "WordPress REST API Handbook",
  "url": "https://developer.wordpress.org/rest-api/",
  "retrieved_at": "2026-03-25T18:00:00Z",
  "fresh_until": "2026-09-25T18:00:00Z",
  "claim_coverage": ["wp scheduling support", "post status semantics"],
  "excerpt_hash": "sha256:..."
}
```

### ValidationResult

```json
{
  "draft_id": "drf_001",
  "claims": [
    {
      "text": "Web designers should plan redirects during a redesign.",
      "risk_type": "general_process_guidance",
      "support_status": "supported",
      "source_ids": ["src_003"]
    }
  ],
  "unsupported_claim_count": 0,
  "hard_override_reasons": [],
  "freshness_status": "pass"
}
```

### ApprovalDecision

```json
{
  "draft_id": "drf_001",
  "decision": "auto_publish",
  "decided_by": "system",
  "usefulness_score": 31,
  "risk_score": 18,
  "reason_codes": ["all_claims_supported", "low_risk_topic", "auto_publish_enabled"]
}
```

## Database Entities

| Table | Purpose | Key Constraints |
| --- | --- | --- |
| `clients` | tenant identity and status | unique `client_id` |
| `client_config_versions` | versioned config snapshots | unique `client_id, version` |
| `wordpress_connections` | tenant WordPress connection settings | unique `client_id` |
| `topic_clusters` | cluster definitions and priorities | unique `client_id, canonical_cluster_key` |
| `editorial_plan_windows` | per-client planned publish windows | unique `client_id, window_start, canonical_topic_key` |
| `post_briefs` | approved briefs | unique `client_id, canonical_topic_key, content_cycle` |
| `draft_packages` | generated content versions | unique `brief_id, version` |
| `source_documents` | normalized source records | unique `client_id, source_hash` |
| `source_ledger_entries` | source references attached to drafts or claims | indexed by `draft_id`, `claim_id` |
| `claim_records` | extracted factual claims | indexed by `draft_id` |
| `validation_runs` | validation outcomes | unique `draft_id, run_version` |
| `usefulness_scores` | usefulness scoring snapshots | unique `draft_id, run_version` |
| `risk_scores` | risk scoring snapshots | unique `draft_id, run_version` |
| `approval_decisions` | system or human decisions | indexed by `draft_id`, `decision_state` |
| `publish_jobs` | queued WordPress jobs | unique `draft_id, publish_attempt_group` |
| `wordpress_taxonomy_terms` | cached WordPress category and tag catalog with usage data | unique `client_id, taxonomy, wordpress_term_id` and unique `client_id, taxonomy, normalized_slug` |
| `wordpress_taxonomy_aliases` | approved alias mappings to live WordPress terms | unique `client_id, taxonomy, alias_normalized` |
| `wordpress_post_links` | internal to WordPress mapping | unique `client_id, wordpress_post_id` and unique `draft_id` |
| `publish_state_history` | publish lifecycle log | indexed by `draft_id`, `changed_at` |
| `review_tasks` | operator review tasks | indexed by `client_id`, `status`, `priority` |
| `refresh_candidates` | refresh and pruning queue | indexed by `client_id`, `reason_code` |
| `incident_log` | operational failures and disputes | indexed by `client_id`, `severity` |

## Queue and Job Design

Job table fields:
- `job_id`
- `job_type`
- `tenant_id`
- `subject_id`
- `status`
- `attempt_count`
- `max_attempts`
- `available_at`
- `claimed_by`
- `claim_expires_at`
- `last_error_code`
- `last_error_detail`
- `payload_json`

Rules:
- workers claim jobs ordered by priority and `available_at`
- transient failures back off exponentially
- permanent failures create an incident and, where relevant, a `review_task`
- no worker may mutate review decisions retroactively; new states append history rows

## Approval Logic

```text
if prohibited_without_confirmation exists:
    decision = blocked
elif unsupported_claim_count > 0:
    decision = blocked
elif freshness_status == fail:
    decision = require_review
elif hard_override_reasons not empty:
    decision = require_review
elif usefulness_score < draft_min_usefulness:
    decision = blocked
elif risk_score >= 60:
    decision = blocked
elif risk_score >= 25:
    decision = require_review
elif usefulness_score >= auto_publish_min_usefulness
  and risk_score <= auto_publish_max_risk
  and client_policy.auto_publish_enabled(topic_type):
    decision = auto_publish
else:
    decision = draft_only
```

Human reviewer actions:
- approve to scheduled
- approve to draft only
- return for revision
- block
- override with reason and evidence reference

## Idempotency and Duplicate Prevention

- Use `client_id + canonical_topic_key + content_cycle` as the primary content identity
- Slugs are deterministic from `canonical_topic_key`, location token, and cycle when needed
- Before creating a WordPress post, check internal mapping first, then confirm whether an existing WordPress post with the same slug belongs to the same content identity
- Retry behavior updates the same WordPress post when mapping exists
- Duplicate-topic detection runs before draft generation and before scheduling

## Security and Audit Requirements

- Store WordPress Application Passwords encrypted at rest via an environment-managed encryption key
- Mask secrets in logs and UI
- Record every approval, override, publish attempt, rollback, and unpublish action with actor, timestamp, and rationale
- Require HTTPS for all WordPress endpoints
- Reject non-HTTPS site URLs unless explicitly allowed in a non-production environment

## Adapter Interfaces

### Research Source Resolver

Responsibilities:
- normalize approved source metadata
- classify tier
- compute freshness windows
- attach evidence to claims

### WordPress Publisher

Responsibilities:
- test auth
- refresh taxonomy cache from WordPress
- resolve categories and tags through exact slug, casefolded name, approved alias, then highest usage count
- require at least one non-blocked category before create or update
- create new categories or tags only when no reusable term exists and the tenant taxonomy policy allows auto-create
- create or update posts
- set `draft` or `future` status
- return WordPress post ID, edit link, and canonical URL when available

### Optional SEO Meta Adapter

Responsibilities:
- sync SEO title, meta description, or custom meta only when the target WordPress site registers compatible fields

The SEO adapter is optional and cannot block core publish flows.

## Phase 1 Deferrals

- Search Console or GA4-driven scoring inputs
- CRM lead-quality feedback
- automated email-derivative publishing
- plugin-specific custom fields as a required path
- high-risk solar policy packs beyond baseline block rules
