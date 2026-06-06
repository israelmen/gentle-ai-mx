# Archive Report: issue-tracking-preflight

**Change**: issue-tracking-preflight
**Archived**: 2026-06-05
**Commit**: `09a7287 feat(sdd): add issue tracking (Group E) to session preflight`
**Stats**: 27 files changed, 595 insertions(+), 74 deletions(-)

---

## Summary

This change adds Issue Tracking as Group E to the SDD Session Preflight process across
all 11 platform orchestrator assets, enhances `sdd-po` with ticket analysis and lifecycle
hooks (`analyze-ticket`, `post-apply-report`, `lifecycle-update`), and updates the Go
migration function (`ensurePreservedOpenCodeOrchestratorPreflight`) to preserve Group E
across sessions.

---

## Verify Status

The `sdd-verify` phase returned **FAIL** due to 3 criticals. All 3 were resolved
post-verify before archiving:

| Critical | Fix Applied |
|----------|-------------|
| Spec used `jira-first` naming | Spec updated: all `jira-first` → `analyze-ticket`; default changed from `github` → `none` |
| Missing `lifecycle-update` post-archive routing | Added to all 11 orchestrators + `skills/sdd-po/SKILL.md` Section F |
| Spec/impl mismatch for `github` routing | Spec updated to route `github`, `jira`, `both` through `sdd-po`; only `none` skips |

All automated tests pass (49 packages, all green).

---

## Delivered

### Phase 1 — Foundation / Contracts (3/3 tasks ✅)

- `internal/assets/skills/_shared/persistence-contract.md` — added `issue_tracking` to both sub-agent context templates (with-deps and no-deps)
- `skills/sdd-po/SKILL.md` — added Section D (`analyze-ticket`), Section E (`post-apply-report`), Section F (`lifecycle-update`); updated What You Receive
- `internal/components/sdd/inject_test.go` — RED tests added for Group E migration (`TestEnsurePreservedPreflightAppendsGroupE`, `TestEnsurePreservedPreflightNoOpWhenGroupEPresent`)

### Phase 2 — Core Implementation (3/3 tasks ✅)

- `internal/components/sdd/inject.go` — Group E guard check + EN/ES preflight block + reply hint + `E1/None → issue_tracking: none` mapping
- `internal/assets/opencode/sdd-orchestrator.md` — full Group E integration: blocks EN/ES, canonical mapping, routing section with `lifecycle-update`
- All 10 non-OpenCode orchestrators (`claude`, `cursor`, `gemini`, `codex`, `antigravity`, `generic`, `windsurf`, `qwen`, `kiro`, `kimi`) — `### Issue Tracking` section added with `analyze-ticket`, `post-apply-report`, `lifecycle-update`

### Phase 3 — Integration / Verification (2/3 tasks ✅ + 1 manual)

- `inject_test.go` GREEN: both Group E migration tests pass
- `assets_test.go` GREEN: `TestOpenCodeSDDOrchestratorHasGroupE` + `TestNonOpenCodeOrchestratorsHaveIssueTrackingQuestion` (10 subtests) pass
- **Task 3.3** (manual): `sdd-po` `analyze-ticket` / `post-apply-report` flows not runtime-verified — **pending manual follow-up** (see below)

### Phase 4 — Cleanup / Review Readiness (1/2 tasks ✅ + 1 manual)

- Golden files regenerated; full `./...` suite clean (49 packages)
- **Task 4.2** (manual): end-to-end spec→code traceability audit — **pending manual follow-up** (see below)

### Post-Verify Fixes (5 tasks ✅)

- `openspec/changes/issue-tracking-preflight/specs/issue-tracking/spec.md` — `jira-first` → `analyze-ticket` (6 occurrences); default `github` → `none`
- All 10 non-OpenCode orchestrators — `lifecycle-update` post-archive routing added
- `internal/assets/opencode/sdd-orchestrator.md` — `lifecycle-update` added to routing table (all 3 tracked values: github, jira, both)
- `skills/sdd-po/SKILL.md` — Section F (`lifecycle-update` mode) added after Section E
- Golden files regenerated (second pass); full `./...` suite passes (49 packages, all green)

---

## Test Coverage

| Package | Coverage |
|---------|----------|
| `internal/components/sdd` | 81.7% |
| `internal/assets` | 63.6% |
| `internal/verify` | 93.9% |

---

## Engram Artifact IDs

| Artifact | Engram ID | Topic Key |
|----------|-----------|-----------|
| Proposal | #864 | `sdd/issue-tracking-preflight/proposal` |
| Spec | #865 | `sdd/issue-tracking-preflight/spec` |
| Design | #866 | `sdd/issue-tracking-preflight/design` |
| Tasks | #867 | `sdd/issue-tracking-preflight/tasks` |
| Apply Progress | #868 | `sdd/issue-tracking-preflight/apply-progress` |
| Verify Report | #869 | `sdd/issue-tracking-preflight/verify-report` |
| Archive Report | (see below) | `sdd/issue-tracking-preflight/archive-report` |

**Note on Engram spec artifact (#865)**: This observation reflects the pre-verify-fix
state (`jira-first`, default `github`). The canonical, post-fix spec is the synced
main spec at `openspec/specs/issue-tracking/spec.md`.

---

## Spec Synced

| Domain | Source | Destination | Action |
|--------|--------|-------------|--------|
| `issue-tracking` | `openspec/changes/issue-tracking-preflight/specs/issue-tracking/spec.md` | `openspec/specs/issue-tracking/spec.md` | **Created** (new domain — no prior spec existed) |

---

## Manual Follow-Ups

These tasks require human interaction with a running `sdd-po` agent and are not
blocking the archive:

### Task 3.3 — Verify `sdd-po` flows

- **analyze-ticket mode**: Run against a real Jira or GitHub ticket. Confirm structured
  context (completeness score + extracted requirements) is returned without overwriting
  existing ticket content.
- **post-apply-report mode**: Run after a real `sdd-apply`. Confirm the test summary
  table is formatted correctly and the ticket's "Technical Solution" section is updated.
  Also confirm graceful handling when `apply-progress` is missing or has unrecognized format.

### Task 4.2 — Spec-to-code traceability audit

- Re-read `design.md` and `tasks.md` for this change.
- Confirm every spec scenario maps to either code, an automated test, or a recorded
  manual check.
- If any scenario is unverified, add a failing test or record a manual verification
  transcript.

---

## Lessons Learned

1. **Spec-first naming alignment**: The `jira-first` naming mismatch (spec said
   `jira-first`; implementation used `analyze-ticket`) was present from the initial spec
   but only surfaced at verify. Reviewing spec mode names against `sdd-po/SKILL.md`
   before apply starts prevents this class of mismatch.

2. **Routing logic needs explicit skip conditions**: The `github` routing ambiguity
   existed from the initial spec. Explicitly stating skip conditions (`none` only)
   vs. trigger conditions (`github`, `jira`, `both`) in routing requirements is less
   ambiguous than describing only the positive case.

3. **Manual verification items need entry criteria**: Tasks 3.3 and 4.2 remain open at
   archive time because "manually verified" was not defined up front (e.g., "record a
   transcript using ticket key X in Jira project Y"). For future cycles, define done
   criteria for manual steps so they can be closed properly.

4. **Golden file regeneration is a two-pass operation**: Original implementation caused
   golden test failures → first regenerate. Post-verify fixes caused golden test failures
   again → second regenerate. This is expected but should be called out explicitly in
   the tasks plan to avoid confusion.

5. **Post-verify fixes should be isolated commits**: The commit `09a7287` bundles the
   original implementation and all post-verify fixes together. Separate commits per fix
   (or a dedicated "fix verify criticals" commit) would make bisection easier if a
   regression is introduced.

---

## SDD Cycle Summary

| Phase | Status | Engram / File |
|-------|--------|---------------|
| explore | ✅ Completed | session context |
| propose | ✅ Completed | #864 / `proposal.md` |
| spec | ✅ Completed (post-verify fix applied) | #865 (outdated) / `openspec/specs/issue-tracking/spec.md` |
| design | ✅ Completed | #866 / `design.md` |
| tasks | ✅ Completed | #867 / `tasks.md` |
| apply | ✅ Completed (14/14 automated tasks) | #868 / — |
| verify | ⚠️ FAIL → post-verify fixes → all criticals resolved | #869 / `verify-report.md` |
| archive | ✅ This report | (below) / `archive-report.md` |

**SDD cycle complete.** Ready for the next change.
