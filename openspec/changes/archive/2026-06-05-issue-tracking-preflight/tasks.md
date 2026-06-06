# Tasks: Issue Tracking Preflight Option & Ticket Enrichment

## Review Workload Forecast

| Field | Value |
|-------|-------|
| Estimated changed lines | 520-760 |
| 400-line budget risk | High |
| Chained PRs recommended | Yes |
| Suggested split | PR1 contracts+skill → PR2 assets+migration → PR3 tests+polish |
| Delivery strategy | ask-always |
| Chain strategy | pending |

Decision needed before apply: Yes
Chained PRs recommended: Yes
Chain strategy: pending
400-line budget risk: High

### Suggested Work Units

| Unit | Goal | Likely PR | Notes |
|------|------|-----------|-------|
| 1 | Define contracts and PO hooks | PR 1 | Prefer feature-branch-chain; base = tracker branch; includes `persistence-contract.md` + `skills/sdd-po/SKILL.md` |
| 2 | Add preflight/routing and migration | PR 2 | Base = PR 1 branch; includes OpenCode + other orchestrator assets + `internal/components/sdd/inject.go` |
| 3 | Lock behavior with tests and polish | PR 3 | Base = PR 2 branch; includes `assets_test.go`, `inject_test.go`, and wording cleanup |

## Phase 1: Foundation / Contracts

- [x] 1.1 Update `internal/assets/skills/_shared/persistence-contract.md` to pass `issue_tracking` in sub-agent context tables and examples.
- [x] 1.2 Extend `skills/sdd-po/SKILL.md` with `analyze-ticket` inputs/outputs, conservative enrichment rules, and `post-apply-report` behavior.
- [x] 1.3 RED: add `internal/components/sdd/inject_test.go` cases for missing-E migration and E-present no-op.

## Phase 2: Core Implementation

- [x] 2.1 Update `internal/components/sdd/inject.go` guard and preserved preflight block to detect and append Group E in EN/ES.
- [x] 2.2 Edit `internal/assets/opencode/sdd-orchestrator.md` to add Group E labels, E1-E4 mapping, cached `issue_tracking`, and Jira-only routing.
- [x] 2.3 Edit `internal/assets/claude/sdd-orchestrator.md` plus `internal/assets/{windsurf,cursor,gemini,kiro,kimi,qwen,codex,antigravity,generic}/sdd-orchestrator.md` to ask issue tracking after delivery strategy and route `sdd-po`.

## Phase 3: Integration / Verification

- [x] 3.1 GREEN: make `internal/components/sdd/inject_test.go` pass by verifying 4-group prompts migrate to 5 groups and current prompts stay unchanged.
- [x] 3.2 Add `internal/assets/assets_test.go` assertions for Group E wording, canonical mappings, and Jira post-apply hook text.
- [ ] 3.3 Manually verify `sdd-po` flows: `analyze-ticket` returns structured context; `post-apply-report` tolerates missing `apply-progress` and formats test summaries when present.

## Phase 4: Cleanup / Review Readiness

- [x] 4.1 REFACTOR: normalize duplicated issue-tracking wording across orchestrator assets; keep EN/ES labels and canonical values identical.
- [ ] 4.2 Re-read `openspec/changes/issue-tracking-preflight/{design.md,tasks.md}` and confirm every spec scenario maps to code, tests, or manual checks.
