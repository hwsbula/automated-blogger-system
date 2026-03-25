# SOPs

## Objective

Provide repeatable operating procedures for onboarding clients, reviewing content, maintaining risk policy, handling factual disputes, and running the system across many tenants.

## SOP 1: Onboard a New Client

Owner:
- ops owner

Steps:
1. Create the client record and capture core business details.
2. Populate the minimum `ClientConfig` fields: business type, vertical, services, locations, offers, voice rules, prohibited claims, proof-backed claims, internal-link targets, CTA rules, approval thresholds, cadence settings, content exclusions, source policy overrides, and WordPress connection.
3. Test the WordPress connection using the dedicated service account and Application Password.
4. Sync the live WordPress category and tag catalog, record blocked generic terms, set the fallback category, and confirm whether auto-create is allowed for categories or tags.
5. Set fallback permissions based on intake completeness.
6. Review prohibited inference categories with the client or internal account owner.
7. Publish the first approved config version and log onboarding completion.

Outputs:
- active client record
- config version `v1`
- WordPress connection verified
- baseline taxonomy catalog and policy
- onboarding QA status

## SOP 2: Run Monthly Planning

Owner:
- strategy owner

Steps:
1. Review active offers, locations, exclusions, taxonomy policy, and any recent client updates.
2. Generate or refresh topic clusters.
3. Remove duplicate or low-value cluster candidates.
4. Allocate topics into monthly windows using cadence rules.
5. Generate briefs for the approved planning batch.
6. Spot-check topics that touch cost, timelines, comparisons, or local rules.

Outputs:
- monthly editorial plan
- brief queue
- flagged review topics

## SOP 3: Review Blocked or Review-Required Content

Owner:
- reviewer

Steps:
1. Open the review task and inspect reason codes, claim records, source ledger, and taxonomy resolution log when present.
2. Confirm whether the issue is unsupported evidence, stale evidence, prohibited inference, usefulness weakness, or taxonomy mismatch.
3. Choose one action: approve scheduled, approve draft only, return for revision, or block.
4. If overriding a prior decision, attach reviewer rationale and source references.
5. Confirm whether similar queued items need the same policy action.

Outputs:
- approval decision
- reviewer notes
- follow-up action for related content if required

## SOP 4: Update Risk Rules or Policy Packs

Owner:
- platform owner with reviewer signoff

Steps:
1. Identify the trigger: repeated incidents, vertical expansion, policy change, or factual dispute.
2. Update the relevant vertical or topic-type rule.
3. Log the change in the decision log and policy-change notes.
4. Re-score any queued or scheduled content affected by the rule update.
5. Pause auto-publish temporarily if the change affects high-risk content classes.

Outputs:
- updated policy pack
- impacted content review list
- change log entry

## SOP 5: Handle a Factual Dispute

Owner:
- reviewer or account lead

Steps:
1. Open an incident record with the disputed claim, post ID, and source references.
2. Determine whether the post is draft, future, or already published.
3. If draft or future, revert to draft immediately.
4. If published, create an unpublish task and mark related queued content for review.
5. Correct or remove the unsupported claim and rerun validation.
6. Document the root cause and rule change if systemic.

Outputs:
- dispute resolution record
- corrected content or takedown action
- root cause note

## SOP 6: Maintain the System Across Many Clients

Owner:
- ops owner

Daily checks:
- review queue age
- publish job failures
- blocked-content volume
- stale-source alerts

Weekly checks:
- tenant credential health
- duplicate-topic incidents
- taxonomy drift or duplicate-term incidents
- schedule-pattern audit for unnatural cadence
- pending refresh candidates

Monthly checks:
- config drift review
- proof-backed claim library updates
- vertical pack review
- incident trend review

## Service-Level Defaults

- intake review: within 2 business days
- blocked-content review: within 1 business day for scheduled items
- credential failure resolution: same business day
- factual dispute triage: immediate for published content

## Escalation Rules

- pause auto-publish for a client after any high-severity factual incident
- pause a vertical pack if the same policy failure repeats across two clients
- escalate to manual review-only if source quality degrades or reviewer backlog exceeds threshold
