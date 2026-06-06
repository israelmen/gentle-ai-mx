# Issue Tracking Specification

## Purpose

Defines required behavior for the SDD Issue Tracking preflight option (Group E),
orchestrator routing to `sdd-po`, and the `sdd-po` agent behaviors for ticket
analysis and lifecycle reporting.

---

## Requirements

### Requirement: Preflight Configuration for Issue Tracking

The system MUST prompt the user for an issue tracking preference (Group E) during preflight initialization.
The canonical values MUST be `github`, `jira`, `both`, or `none`.
The default value SHOULD be `none`.
The system MUST cache this preference using the existing caching mechanism for Groups A-D.
The orchestrator MUST propagate the `issue_tracking` value to sub-agents that require it.

#### Scenario: User selects Jira for issue tracking

- GIVEN a new SDD initialization
- WHEN the preflight prompt asks for issue tracking preference
- AND the user selects `jira`
- THEN the system caches `jira` as the `issue_tracking` value
- AND subsequent agent runs receive `issue_tracking: jira` in their context

#### Scenario: User skips selection to use default

- GIVEN a new SDD initialization
- WHEN the preflight prompt asks for issue tracking preference
- AND the user leaves it empty or skipped
- THEN the system caches `none` as the default `issue_tracking` value

---

### Requirement: Orchestrator Routing for Issue Tracking

The orchestrator MUST evaluate the `issue_tracking` value to conditionally trigger `sdd-po` hooks.
If the value is `none`, the orchestrator MUST skip all `sdd-po` delegations.
If the value is `github`, `jira`, or `both`, the orchestrator MUST delegate to `sdd-po` with `mode: analyze-ticket` during the pre-propose phase (if a ticket/issue reference was provided).
If the value is `github`, `jira`, or `both`, the orchestrator MUST delegate to `sdd-po` with `mode: post-apply-report` after the apply phase.
If the value is `github`, `jira`, or `both`, the orchestrator MUST delegate to `sdd-po` with `mode: lifecycle-update` after the archive phase.

#### Scenario: Routing with ticket integration enabled

- GIVEN `issue_tracking` is set to `github`, `jira`, or `both`
- WHEN the orchestrator reaches the respective SDD phases
- THEN the orchestrator delegates to `sdd-po` with `mode: analyze-ticket` pre-propose (if a ticket/issue reference was provided)
- AND delegates to `sdd-po` with `mode: post-apply-report` post-apply
- AND delegates to `sdd-po` with `mode: lifecycle-update` post-archive

#### Scenario: Routing with no integration

- GIVEN `issue_tracking` is set to `none`
- WHEN the orchestrator reaches the SDD phases
- THEN the orchestrator skips all `sdd-po` delegations

---

### Requirement: Ticket Enrichment (analyze-ticket mode)

When running in `analyze-ticket` mode, the `sdd-po` agent MUST read the target ticket (Jira ticket or GitHub issue, depending on `issue_tracking`) and evaluate its completeness.
Completeness criteria MUST include description quality, presence of acceptance criteria, and scope clarity.
The agent MUST output structured requirement context for the `sdd-propose` phase.
The agent MUST NOT overwrite existing ticket content; it MAY only ADD structured data.

#### Scenario: Enriching an incomplete ticket

- GIVEN the `sdd-po` agent is running in `analyze-ticket` mode
- WHEN the target ticket lacks clear acceptance criteria
- THEN the agent evaluates it as incomplete
- AND the agent adds structured acceptance criteria to the ticket without overwriting existing description
- AND the agent outputs the structured requirement context

#### Scenario: Processing a complete ticket

- GIVEN the `sdd-po` agent is running in `analyze-ticket` mode
- WHEN the target ticket already has clear scope and acceptance criteria
- THEN the agent skips enrichment
- AND the agent outputs the structured requirement context as-is

---

### Requirement: Post-Apply Test Report (post-apply-report mode)

When running in `post-apply-report` mode, the `sdd-po` agent MUST read the `apply-progress` artifact for test results.
The agent MUST format the test report including pass/fail status, test names, and test coverage if available.
The agent MUST update the target ticket's "Technical Solution" section with the formatted test report.
The agent MUST gracefully handle missing test output or unrecognized formats without crashing.

#### Scenario: Successfully appending test results

- GIVEN the `sdd-po` agent is running in `post-apply-report` mode
- WHEN the `apply-progress` artifact contains successful test results
- THEN the agent formats the results into a report
- AND the agent updates the "Technical Solution" section of the ticket with the report

#### Scenario: Handling missing test results

- GIVEN the `sdd-po` agent is running in `post-apply-report` mode
- WHEN the `apply-progress` artifact is missing or has unrecognized format
- THEN the agent logs a warning gracefully
- AND the ticket is updated with a note indicating test results were unavailable
