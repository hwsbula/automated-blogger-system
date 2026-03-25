# Failure Modes

## Objective

Document the high-probability and high-impact failure cases for the blogging system, along with detection, recovery, fallback behavior, and ownership.

## Failure Catalog

| Severity | Failure Mode | Detection | Owner | Recovery | Fallback |
| --- | --- | --- | --- | --- | --- |
| high | Unsupported claim survives into validation | validation result shows unsupported claim count above zero | validation owner | block the draft, open review task, require evidence or rewrite | no WordPress action |
| high | Prohibited inference attempted for pricing, guarantees, savings, certifications, warranties, legal, compliance, or hard performance claims | claim classifier matches prohibited category without proof | validation owner | block and log policy breach | no draft or schedule creation |
| high | Stale authoritative source on compliance or incentive topic | freshness checker fails required source | validation owner | downgrade to review or block until fresh source is attached | no auto-publish |
| high | Wrong client config applied to a draft | tenant mismatch during job processing or review | platform owner | stop job, create incident, re-run under correct tenant | freeze affected tenant queue |
| high | WordPress credentials invalid or revoked | `401` or `403` from REST API | integration owner | disable publish jobs for tenant and open credential refresh task | hold content in internal queue |
| high | Duplicate slug collision with mismatched content identity | internal mapping check plus WordPress slug lookup conflict | integration owner | route to review, choose merge, rename, or canonical redirect path | do not create second post |
| medium | Taxonomy term missing and auto-create disabled | publish adapter cannot resolve configured category or tag | integration owner | open review task to map or create term | keep post internal or draft-only |
| medium | Duplicate or low-value taxonomy term is auto-created because the cache or alias map is stale | new term slug or normalized name overlaps an existing live term after publish | integration owner | merge mapping, add alias rule, refresh tenant taxonomy cache, and pause tenant auto-create if repeated | route affected posts to manual taxonomy review |
| medium | Review backlog exceeds service-level target | queue age metrics cross threshold | ops owner | pause auto-publish for affected tenant or topic class, reassign reviewers | create draft-only instead of schedule where safe |
| medium | Draft usefulness score below threshold | scoring service returns under 24 | strategy owner | return for re-brief or discard topic | no publish action |
| medium | Duplicate-topic cluster planned twice | duplicate detector on `canonical_topic_key` or semantic similarity threshold | strategy owner | merge or defer topic | do not generate second brief |
| medium | Worker crashes mid-job | heartbeat expires on claimed job | platform owner | release claim after timeout and requeue if safe | create incident after max retries |
| medium | Schedule drift causes unnatural clustering | cadence audit flags repeat pattern or excessive bunching | strategy owner | recompute publish windows | convert some items to draft-only until windows rebalance |
| low | Optional SEO adapter fails | adapter error after core WordPress post update | integration owner | log adapter incident and continue | core publish still succeeds |
| low | Client config missing non-critical voice preference | config completeness warning | ops owner | mark advisory gap and continue within defaults | proceed with conservative default voice rules |

## Detection Rules

- Validate every draft before any publish job is queued
- Run freshness checks during validation and again before scheduled publish
- Reconfirm WordPress credential health before the first publish job of each worker cycle
- Refresh the tenant taxonomy cache before the first publish job of each worker cycle and after any term auto-create
- Run duplicate-topic detection during planning and again before publish
- Monitor queue age, retry count, and blocked-item volume daily

## Retry Policy

Retry automatically only when the failure is transient:
- network timeout
- rate limit
- temporary server error
- worker crash without evidence of partial side effects

Do not auto-retry:
- policy breaches
- unsupported claims
- tenant mismatch
- duplicate slug collision
- invalid credentials
- stale required sources

## Rollback and Unpublish Procedure

### Scheduled or Draft State

1. Change the internal state to `rollback_requested`.
2. Update the WordPress post to `draft`.
3. Append a `publish_state_history` row with actor, reason, and previous state.
4. Open a review task if the content may be rescheduled.

### Published State

1. Create an incident with severity based on claim or brand risk.
2. Generate an unpublish task for a human operator.
3. Record the current WordPress URL, publish time, and reason for takedown.
4. Pause related cluster or claim class auto-publish rules until root cause review completes.

## Root Cause Review Requirements

Every high-severity incident must answer:
- what failed
- why the existing gate did not stop it
- whether a config, source-policy, or scoring rule must change
- whether any already-scheduled content shares the same risk

## Backlog Thresholds

- review queue age over 3 business days: alert and consider pausing auto-publish for affected tenant
- blocked-content rate over 20 percent in a monthly batch: review planning and source policy
- publish failure rate over 5 percent in a rolling week: inspect credentials, taxonomy mappings, and retry classification
- more than one duplicate taxonomy incident in a tenant month: disable tenant auto-create and review alias rules
- stale-source count above 10 percent of scheduled inventory: run refresh campaign before new content generation
