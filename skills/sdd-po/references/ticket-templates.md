# Jira Ticket Templates

Templates for PO ticket creation. Select based on issue type.

---

## Requirement (default)

```markdown
## User Story
As a [role], I want [goal], so that [benefit].

## Description
[Detailed description from proposal intent + approach]

## Acceptance Criteria
- [ ] [From proposal Success Criteria]
- [ ] [From proposal Capabilities]

## Test Cases
### Happy Path
- Given [precondition], when [action], then [expected result]

### Edge Cases
- Given [edge condition], when [action], then [expected result]

### Error Scenarios
- Given [error condition], when [action], then [expected handling]

## Scope
### In Scope
- [From proposal]

### Out of Scope
- [From proposal]

## Priority
[Derived from proposal risk assessment]

## Labels
sdd, {change-name}
```

---

## Bug

```markdown
## Summary
[Bug description]

## Steps to Reproduce
1. [Step]
2. [Step]

## Expected Behavior
[What should happen]

## Actual Behavior
[What happens instead]

## Test Cases
### Regression
- Given [fixed condition], when [original trigger], then [correct behavior]

## Priority
[From severity]

## Labels
sdd, bug, {change-name}
```

---

## Tech Debt

```markdown
## Summary
[What technical debt is being addressed]

## Motivation
[Why this matters — impact on maintainability, performance, etc.]

## Acceptance Criteria
- [ ] [Measurable improvement]

## Test Cases
### Validation
- Given [refactored component], when [existing functionality], then [behavior unchanged]

## Priority
[From risk assessment]

## Labels
sdd, tech-debt, {change-name}
```
