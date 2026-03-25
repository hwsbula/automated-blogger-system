# QA Checklist

## Objective

Validate that the blogging system artifacts, system design, and handoff materials are internally consistent, operationally safe, and ready for implementation.

## Level 1: Artifact QA

### Project Brief

- [ ] Objective matches the requested business outcome
- [ ] Route selection is `R-HYBRID` and rationale is explicit
- [ ] Workstreams and success criteria are specific
- [ ] Scope boundaries and phase 1 constraints are visible

### System Architecture

- [ ] All nine pipeline stages are present and ordered correctly
- [ ] Generation is separate from validation and approval
- [ ] Source policy tiers are explicit
- [ ] Fallback policy distinguishes allowed and prohibited inference
- [ ] Risk and usefulness thresholds are defined
- [ ] WordPress is the only required direct integration in phase 1

### Technical Spec

- [ ] `ClientConfig` schema includes all required fields
- [ ] Core record types cover planning, validation, publishing, and audit needs
- [ ] Taxonomy policy, cache tables, and alias mapping are defined
- [ ] Approval logic is deterministic
- [ ] Idempotency and duplicate-prevention rules are explicit
- [ ] Retry behavior distinguishes transient from permanent failure

### Integrations

- [ ] Authentication model is production-safe
- [ ] Draft and scheduled publish behaviors are defined
- [ ] Category, tag, slug, and featured-image placeholder rules are explicit
- [ ] Existing live WordPress categories and tags are checked before any auto-create path
- [ ] At least one non-blocked category is required, and zero-tag cases require a recorded reason
- [ ] Plugin-agnostic default is preserved

### Workflow and Failure Docs

- [ ] Workflow map aligns with approval-state definitions
- [ ] Failure modes include detection, owner, recovery, and fallback
- [ ] Rollback and unpublish procedures are present

### Operations Docs

- [ ] SOPs are executable by an operator
- [ ] QA checklist includes pass, revise, and block outcomes
- [ ] Handoffs include ownership and next actions

## Level 2: System QA

- [ ] Workstream boundaries are clear and non-overlapping
- [ ] Phase 1 safe scope is explicit
- [ ] Deferred integrations are isolated and non-blocking
- [ ] High-risk verticals are not treated as launch defaults
- [ ] Auto-publish can be disabled globally, by client, or by vertical
- [ ] Monitoring covers publish failures, validation failures, duplicate topics, stale sources, review backlog, and rollback events
- [ ] Monitoring covers duplicate taxonomy creation and stale taxonomy cache incidents
- [ ] The system remains usable with weak client inputs under the documented fallback policy

## Level 3: Handoff QA

- [ ] A new operator can identify what to review first
- [ ] Open decisions are visible
- [ ] Ownership for review, publishing, and incidents is explicit
- [ ] Next actions are implementation-usable
- [ ] No hidden assumptions remain about architecture, risk, cost, or measurement

## QA Outcomes

### Pass

Use when all core requirements are present and no blocker affects implementation safety.

### Revise

Use when the system is directionally correct but one or more docs still leave material ambiguity around thresholds, ownership, or failure handling.

### Block

Use when unsupported claims could slip through, WordPress behavior is underspecified, or handoff materials require the next operator to infer core architecture.
