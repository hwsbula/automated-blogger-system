# Threading Decision

## Objective

Determine whether the blogging-system design should remain in one thread or be split into multiple workstreams.

## Route ID

R-HYBRID

## Decision

Multiple workstreams

## Reasoning

- Strategy and technical execution are both first-order outcomes.
- The project requires more than six material artifacts across architecture, build, automations, operations, and final handoff.
- Validation policy, WordPress publishing logic, and operating procedures need bounded ownership to stay clear.
- QA should remain separate from production design because claim safety and approval logic are release-critical.
- The route standard marks `R-HYBRID` as a mandatory multi-workstream case.

## Workstreams

- Master systems engineer
- Technical automation architect
- Content strategy and validation architect
- WordPress integration owner
- Operations and SOP owner
- QA reviewer

## Risks

- Cross-workstream inconsistency between validation policy and publishing behavior
- Ambiguity around proof thresholds for higher-risk verticals if assumptions are not logged
- Review backlog if approval rules are stricter than operational staffing can support

## Review Trigger

Revisit this decision if the project scope is reduced to documentation only, or if later phases add non-WordPress data integrations that require separate reporting or analytics workstreams.
