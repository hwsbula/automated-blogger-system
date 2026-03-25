# Handoffs

## Objective

Define how work moves between system owners without losing audit context, approval status, or operational clarity.

## Ownership Matrix

| Area | Primary Owner | Reviewer or Backup | Main Outputs | Acceptance Condition |
| --- | --- | --- | --- | --- |
| client intake and config | ops owner | QA reviewer | client record, config version, connection status | config is complete enough to plan safely |
| strategy and cluster planning | strategy owner | reviewer for sensitive topics | topic clusters, editorial windows, briefs | duplicate checks and exclusion checks pass |
| draft generation | draft pipeline owner | validation owner | draft package | draft obeys brief and prohibited-claim rules |
| validation and scoring | validation owner | QA reviewer | source ledger, claim records, usefulness and risk scores | decision state can be assigned deterministically |
| review queue | reviewer | ops owner | approval decision, notes, override rationale | next state is clear and logged |
| WordPress publishing | integration owner | ops owner | WordPress post, publish-state history, incident if any | post is created or updated idempotently |
| refresh and pruning | strategy owner | account lead | refresh brief, prune recommendation | stale or off-strategy content has a documented action |

## Required Handoff Packet

Every handoff must include:
- objective
- current state
- client or tenant ID
- files or records to review first
- open decisions
- risks or blockers
- next required action
- QA status

## Handoff Types

### 1. Onboarding to Planning

Packet contents:
- approved config version
- intake completeness score
- fallback permissions
- WordPress connection status
- proof-backed and prohibited claim library

Receiver goal:
- generate a safe topic plan without inventing restricted claims

### 2. Planning to Drafting

Packet contents:
- approved topic cluster
- publish window
- post brief
- taxonomy hints
- required sources
- blocked claims reminder

Receiver goal:
- generate a draft that is structurally ready for validation

### 3. Validation to Review

Packet contents:
- validation result
- usefulness score
- risk score
- hard override reasons
- reviewer recommendation

Receiver goal:
- approve, revise, or block without re-investigating the full tenant history

### 4. Review to Publishing

Packet contents:
- final decision state
- approved schedule window or draft-only instruction
- resolved primary category
- resolved secondary category if used
- resolved tags or recorded no-tag reason
- taxonomy reuse-versus-create decision log
- slug decision
- reviewer notes

Receiver goal:
- create or update the correct WordPress post without duplicate, schedule, or taxonomy drift errors

### 5. Incident to Operations

Packet contents:
- incident severity
- affected post IDs
- trigger event
- current publish state
- recommended rollback or remediation action

Receiver goal:
- stabilize the tenant, protect against repeated failure, and document root cause

### 6. Refresh Loop to Strategy

Packet contents:
- refresh candidate record
- stale-source reasons
- offer or CTA drift notes
- duplicate or cannibalization notes

Receiver goal:
- decide whether to refresh, consolidate, archive, or prune

## Review-First Files

For a new operator, review in this order:
1. `01-foundation/project-brief.md`
2. `02-system-design/system-architecture.md`
3. `02-system-design/technical-spec.md`
4. `03-workflows/failure-modes.md`
5. `04-operations/sops.md`

## Handoff Rule

The receiving operator should be able to answer immediately:
- what state the content or client is in
- why it is in that state
- what can happen next
- what must not be changed without review
