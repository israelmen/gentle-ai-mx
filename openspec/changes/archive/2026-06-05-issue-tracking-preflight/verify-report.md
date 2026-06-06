## Verification Report

**Change**: issue-tracking-preflight
**Version**: N/A
**Mode**: Strict TDD (`openspec/config.yaml` sets `strict_tdd: true`; Go test runner available)

### Completeness
| Metric | Value |
|--------|-------|
| Tasks total | 11 |
| Tasks complete | 9 |
| Tasks incomplete | 2 |

### Build & Tests Execution
**Build / Type Check**: ✅ Passed
```text
go vet ./...
(no output)
```

**Tests**: ✅ Full suite passed
```text
go test ./... -count=1
PASS across all packages; no failures reported.
```

**Coverage**: ✅ Available
```text
go test -cover ./...
- internal/components/sdd: 81.7%
- internal/assets: 63.6%
- internal/verify: 93.9%
```

### Previous Criticals Status
| Previous Critical | Status | Notes |
|---|---|---|
| Spec used `jira-first` and default `github` | ⚠️ PARTIALLY RESOLVED | Spec now uses `analyze-ticket` and default `none`, but routing still conflicts with spec for `github` (spec says skip all `sdd-po` delegations for `github`/`none`; all orchestrators still route `github` through `sdd-po`). |
| Missing `lifecycle-update` post-archive routing | ✅ RESOLVED | Present in OpenCode plus all 10 non-OpenCode orchestrators; `skills/sdd-po/SKILL.md` now has Section F for `lifecycle-update`. |
| Non-OpenCode separate-question pattern flagged as defect | ✅ DISMISSED (FALSE POSITIVE) | Separate-question pattern is intentional for non-OpenCode orchestrators; no defect confirmed. |

### TDD Compliance
| Check | Result | Details |
|-------|--------|---------|
| TDD Evidence reported | ✅ | `apply-progress` includes a TDD Cycle Evidence table |
| RED confirmed (tests exist) | ✅ | `inject_test.go`, `assets_test.go`, and golden tests exist and executed |
| GREEN confirmed (tests pass) | ✅ | Full suite, vet, and coverage commands passed |
| Triangulation adequate | ⚠️ | Spec scenarios for `analyze-ticket` / `post-apply-report` still have no passing runtime coverage |
| Safety Net for modified files | ⚠️ | No explicit safety-net evidence recorded in `apply-progress` |

**TDD Compliance**: 3/5 checks passed

### Test Layer Distribution
| Layer | Tests | Files | Tools |
|-------|-------|-------|-------|
| Unit / asset / golden | 16+ named tests (plus subtests) | 3+ | `go test` |
| Integration | 0 | 0 | not used |
| E2E | 0 | 0 | not used |
| **Total** | **16+** | **3+** | |

### Changed File Coverage
| File | Coverage Evidence | Notes |
|------|-------------------|-------|
| `internal/components/sdd/inject.go` | ✅ Covered by `internal/components/sdd` package tests (81.7% pkg coverage) | Group E migration tests passed |
| `internal/assets/assets_test.go` | ✅ Covered by `internal/assets` package tests (63.6% pkg coverage) | Validates issue-tracking text presence |
| `internal/components/golden_test.go` | ✅ Covered by full suite | Golden regeneration validated current orchestrator text |
| `skills/sdd-po/SKILL.md` | N/A | Markdown skill file; inspected statically only |
| `internal/assets/skills/_shared/persistence-contract.md` | N/A | Markdown contract; inspected statically only |

### Assertion Quality
**Assertion quality**: ✅ All inspected changed tests verify real behavior; no tautologies or ghost-loop patterns found.

### Quality Metrics
**Linter**: ➖ Not configured
**Type Checker**: ✅ `go vet ./...` passed

### Spec Compliance Matrix
| Requirement | Scenario | Evidence | Result |
|-------------|----------|----------|--------|
| Preflight Configuration for Issue Tracking | User selects Jira for issue tracking | `TestEnsurePreservedPreflightAppendsGroupE`; `TestOpenCodeSDDOrchestratorHasGroupE`; `TestNonOpenCodeOrchestratorsHaveIssueTrackingQuestion`; full suite pass | ⚠️ PARTIAL |
| Preflight Configuration for Issue Tracking | User skips selection to use default | Spec updated to `none`; OpenCode `E1 None (recommended)`; non-OpenCode assets mark `none` as default; full suite pass | ⚠️ PARTIAL |
| Orchestrator Routing for Issue Tracking | Routing with Jira enabled | All 11 orchestrators include `analyze-ticket`, `post-apply-report`, `lifecycle-update`; full suite pass | ⚠️ PARTIAL |
| Orchestrator Routing for Issue Tracking | Routing with GitHub or None enabled | Spec requires skipping all `sdd-po` delegations for `github` or `none`; implementation still routes `github` through `analyze-ticket`, `post-apply-report`, and `lifecycle-update` in all 11 orchestrators | ❌ FAILING |
| Ticket Enrichment (`analyze-ticket` mode) | Enriching an incomplete ticket | `skills/sdd-po/SKILL.md` Section D exists, but no passing runtime/manual verification was recorded | ❌ UNTESTED |
| Ticket Enrichment (`analyze-ticket` mode) | Processing a complete ticket | `skills/sdd-po/SKILL.md` Section D exists, but no passing runtime/manual verification was recorded | ❌ UNTESTED |
| Post-Apply Test Report (`post-apply-report` mode) | Successfully appending test results | `skills/sdd-po/SKILL.md` Section E exists, but no passing runtime/manual verification was recorded | ❌ UNTESTED |
| Post-Apply Test Report (`post-apply-report` mode) | Handling missing test results | `skills/sdd-po/SKILL.md` graceful-degradation rules exist, but no passing runtime/manual verification was recorded | ❌ UNTESTED |

**Compliance summary**: 0/8 scenarios fully compliant, 3/8 partial, 1/8 failing, 4/8 untested

### Correctness (Static + Runtime Evidence)
| Check | Status | Notes |
|------|--------|-------|
| `analyze-ticket` used in updated spec | ✅ | No `jira-first` remains in `spec.md` |
| No leftover `jira-first` in implementation/docs tied to this change | ⚠️ | `skills/sdd-po/SKILL.md`, `design.md`, and `proposal.md` still contain `jira-first` references |
| Default `none` in spec and orchestrator assets | ✅ | Spec updated; OpenCode uses `E1 None (recommended)`; non-OpenCode assets mark `none` as default |
| `issue_tracking` propagated in shared persistence contract | ✅ | Present in `internal/assets/skills/_shared/persistence-contract.md` |
| Group E migration in `inject.go` | ✅ | Guard/test evidence present in `inject.go` and `inject_test.go` |
| All 11 orchestrators mention `lifecycle-update` post-archive | ✅ | Verified across OpenCode + 10 non-OpenCode orchestrators |
| `skills/sdd-po/SKILL.md` Section F exists | ✅ | `## F. Lifecycle Update Mode (\`lifecycle-update\`)` present |
| Spec routing matches implementation for `github` and `none` | ❌ | Spec says `github`/`none` skip all Jira delegations; implementation routes `github` through `sdd-po` |

### Design Coherence
| Decision | Followed? | Notes |
|----------|-----------|-------|
| Add Group E to OpenCode preserved preflight block | ✅ Yes | Implemented and tested |
| Canonical values `github|jira|both|none` | ✅ Yes | Present in spec and orchestrator assets |
| Add `analyze-ticket` and `post-apply-report` hooks to `sdd-po` | ✅ Yes | Implemented in `skills/sdd-po/SKILL.md` |
| Add post-archive `lifecycle-update` hook | ✅ Yes | Implemented in orchestrators and `sdd-po` Section F |
| Keep design/proposal/spec naming aligned | ⚠️ No | `design.md`, `proposal.md`, and `skills/sdd-po/SKILL.md` still reference `jira-first` |
| Route only `jira` / `both` through `sdd-po` per spec | ❌ No | All orchestrators still route `github` through `sdd-po` |

### Issues Found
**CRITICAL**:
- Spec/implementation mismatch remains: `spec.md` requires skipping all `sdd-po` delegations when `issue_tracking` is `github` or `none`, but all 11 orchestrators still route `github` through `analyze-ticket`, `post-apply-report`, and `lifecycle-update`.
- Required verification is still incomplete for core behavior: task `3.3` remains unchecked, and there is no passing runtime/manual evidence for the `analyze-ticket` or `post-apply-report` scenarios. Under `sdd-verify` rules, untested spec scenarios are not compliant.

**WARNING**:
- Naming consistency is still not clean: `jira-first` remains in `skills/sdd-po/SKILL.md`, `openspec/changes/issue-tracking-preflight/design.md`, and `openspec/changes/issue-tracking-preflight/proposal.md`.
- Cleanup task `4.2` remains incomplete.
- Asset tests enforce presence of issue-tracking text, but not the spec-critical distinction that `github` should skip all `sdd-po` delegations.

**SUGGESTION**:
- Align `proposal.md`, `design.md`, and `skills/sdd-po/SKILL.md` on one pre-propose term: `analyze-ticket`.
- Add executable verification for the `sdd-po` flows, or record a real manual verification transcript for `analyze-ticket` and `post-apply-report`.
- Add tests that explicitly fail if any orchestrator routes `github` through `sdd-po` while the spec says to skip it.

### Verdict
FAIL

The re-verification closed the `lifecycle-update` gap and fixed the spec's `analyze-ticket`/default-`none` wording, but the change still FAILS because the implementation routes `github` through `sdd-po` against the current spec, and the new `sdd-po` behaviors still lack passing runtime verification.
