# Proposal: Issue Tracking Preflight Option & Ticket Enrichment

## Intent

Add an "Issue Tracking" option (group E) to the SDD Session Preflight process to configure GitHub/Jira tracking during the SDD lifecycle. Additionally, enhance the `sdd-po` skill to analyze and proactively enrich existing Jira tickets before feeding them into the SDD pipeline, and automatically update Jira tickets with test reports after the apply phase.

## Scope

### In Scope
- Add group E (Issue Tracking) to all 11 platform orchestrator files.
- Add English + Spanish localization (OpenCode compact block).
- Add mapping to canonical values (`github`, `jira`, `both`, `none`).
- Add caching and propagation instructions (pass `issue_tracking` to sub-agents).
- Update Go migration function in `internal/components/sdd/inject.go`.
- Update `persistence-contract.md` with `issue_tracking` parameter.
- Update `assets_test.go` and `inject_test.go` tests.
- Modify `skills/sdd-po/SKILL.md` to add a "Ticket Analysis & Enrichment" step in jira-first mode to proactively enrich vague tickets with structured requirements before `sdd-propose`.
- Modify `skills/sdd-po/SKILL.md` to add a "Post-apply Test Report" lifecycle hook to collect test results after `sdd-apply` and update the Jira ticket's "Technical Solution" section.

### Out of Scope
- Modifying skill files (`issue-creation`, `branch-pr`).
- Adding new skills.
- Changing the Go adapter interface or TUI.
- Adding Jira MCP server configuration.

## Capabilities

### New Capabilities
- None

### Modified Capabilities
- `sdd-orchestrator-assets`: The orchestrator assets for all 11 platforms will add a new preflight configuration question/block for "Issue Tracking" (Group E) and instructions to propagate this state to sub-agents. It will also be updated to delegate to `sdd-po` with mode `post-apply-report` if `issue_tracking` includes jira after the `sdd-apply` phase.
- `sdd-po`: The product owner skill will be enhanced with new workflow hooks:
  1. Ticket Analysis & Enrichment (Pre-propose): In jira-first mode, evaluate and enrich vague tickets with structured requirement data before passing context to `sdd-propose`.
  2. Post-apply Test Report (Post-apply): After `sdd-apply` completes, read `apply-progress` artifacts to format a test report (pass/fail, coverage) and update the Jira ticket's "Technical Solution" section.

## Approach

Add the group E block to `internal/assets/opencode/sdd-orchestrator.md` with EN/ES localization and mapping logic. Duplicate the equivalent question block to the other 10 platform orchestrator assets. Add `issue_tracking` to the sub-agent state cache rules and update `persistence-contract.md`. Update the migration logic in `internal/components/sdd/inject.go` to support group E and update related tests. Update orchestrator logic to trigger `sdd-po` with `mode: post-apply-report` when apply completes and jira is enabled. Update `skills/sdd-po/SKILL.md` to insert the analysis step before `sdd-propose`, and add the new lifecycle step for the post-apply test reporting to the "Technical Solution" Jira section.

## Affected Areas

| Area | Impact | Description |
|------|--------|-------------|
| `internal/assets/*/sdd-orchestrator.md` | Modified | Add Group E for issue tracking; add post-apply `sdd-po` hook |
| `internal/components/sdd/inject.go` | Modified | Update `ensurePreservedOpenCodeOrchestratorPreflight` to preserve Group E |
| `internal/assets/skills/_shared/persistence-contract.md` | Modified | Add `issue_tracking` to sub-agent context |
| `internal/assets/assets_test.go` | Modified | Update wording tests for group E |
| `internal/components/sdd/inject_test.go` | Modified | Update migration tests for group E preservation |
| `skills/sdd-po/SKILL.md` | Modified | Add ticket analysis and post-apply test report hooks |

## Risks

| Risk | Likelihood | Mitigation |
|------|------------|------------|
| 11 orchestrator files out of sync | Medium | Apply template changes uniformly and verify with `assets_test.go` |
| Go migration function breaks | High | Update `inject_test.go` and ensure regex handles 4 or 5 groups properly |
| Enrichment quality on vague tickets | Medium | Provide strict prompts in `sdd-po` for conservative, structured enrichment |
| Jira API limitations for updates | Low | Delegate to the robust, existing Jira skill for all API operations |
| Test results format varies; output missing | Medium | Make `sdd-po` gracefully handle missing/unrecognized apply-progress test output |

## Rollback Plan

Revert the commits introducing Group E to the orchestrator assets, `inject.go`, test files, and the modifications to `skills/sdd-po/SKILL.md`. The older versions of the assets will be bundled and `inject.go` will revert to handling 4 groups.

## Dependencies

- None (Jira integration already exists in `sdd-po` and `jira` skill)

## Success Criteria

- [ ] Group E is present in all 11 orchestrator assets
- [ ] `issue_tracking` is correctly added to sub-agent state passing rules
- [ ] OpenCode preflight compact block migration preserves Group E selections across sessions
- [ ] `sdd-po` correctly identifies and enriches vague Jira tickets before passing them to `sdd-propose`
- [ ] Jira ticket has test report in "Technical Solution" section after apply phase
- [ ] All `assets_test.go` and `inject_test.go` tests pass