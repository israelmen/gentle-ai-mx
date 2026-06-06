# Design: Issue Tracking Preflight Option & Ticket Enrichment

## Technical Approach

Add group E to the existing A–D preflight compact block (OpenCode) and separate-question sections (other platforms). Extend `ensurePreservedOpenCodeOrchestratorPreflight` to include group E in the migration block. Enhance `sdd-po` with two new hooks: pre-propose ticket analysis and post-apply test reporting. The orchestrator routes to `sdd-po` based on the cached `issue_tracking` value.

## Architecture Decisions

| Decision | Alternatives | Rationale |
|----------|-------------|-----------|
| Group E as last preflight group (after D/Review) | Inline with artifact store; separate post-preflight question | Keeps the A–E sequential pattern. Issue tracking is a session-scoped config like the others. |
| Canonical values: `github`, `jira`, `both`, `none` | Boolean `jira_enabled` | Mirrors existing multi-option pattern (B1–B3). Supports future GitHub Issues integration. |
| sdd-po pre-hook as orchestrator routing (not sdd-propose internal) | Embed analysis inside sdd-propose | sdd-po is already a separate skill; keeping enrichment there preserves single responsibility. Orchestrator decides whether to call it based on `issue_tracking`. |
| Post-apply report reads `apply-progress` artifact | Parse test runner stdout directly | `apply-progress` already exists in the artifact pipeline. Consistent with how sdd-verify reads results. |
| Graceful degradation: skip report section when data missing | Fail loudly on missing test data | Test output format varies; blocking the pipeline on missing coverage data is worse than a partial report. |

## Data Flow

### Pre-propose enrichment (jira-first)

```
Preflight(E=jira) → Orchestrator → sdd-po(mode:analyze-ticket)
                                        │
                                        ├─ Read Jira ticket via Jira skill
                                        ├─ Score completeness
                                        └─ Return enriched context
                                              │
                                    Orchestrator → sdd-propose(enriched_context)
```

### Post-apply reporting

```
sdd-apply completes → Orchestrator checks issue_tracking includes jira
                         │
                         └─ sdd-po(mode:post-apply-report)
                              ├─ Read apply-progress artifact
                              ├─ Extract test results (pass/fail, names, coverage)
                              └─ Update Jira ticket "Technical Solution" section
```

## File Changes

| File | Action | Description |
|------|--------|-------------|
| `internal/assets/opencode/sdd-orchestrator.md` | Modify | Add E group (EN/ES) to compact preflight block; add mapping rules; add `issue_tracking` to cache/propagation; add post-apply sdd-po routing |
| `internal/assets/claude/sdd-orchestrator.md` | Modify | Add issue tracking question after delivery strategy section; add post-apply sdd-po routing |
| `internal/assets/{windsurf,cursor,gemini,kiro,kimi,qwen,codex,antigravity,generic}/sdd-orchestrator.md` | Modify | Same pattern as claude — add separate issue tracking question + routing |
| `internal/components/sdd/inject.go` | Modify | Extend `ensurePreservedOpenCodeOrchestratorPreflight` — add group E to the `preflight` string literal and update the content-detection guard to include an E-group marker |
| `internal/assets/skills/_shared/persistence-contract.md` | Modify | Add `issue_tracking` to sub-agent context passing rules table |
| `skills/sdd-po/SKILL.md` | Modify | Add section A2 (Ticket Analysis & Enrichment) for pre-propose hook; add section D (Post-Apply Test Report) |
| `internal/assets/assets_test.go` | Modify | Add assertion for group E wording presence |
| `internal/components/sdd/inject_test.go` | Modify | Add test case for migration preserving group E |

## Interfaces / Contracts

### Preflight group E (OpenCode compact block)

```text
E. Issue Tracking
   E1 None (recommended): no ticket integration.
   E2 GitHub: create/update GitHub issues.
   E3 Jira: create/update Jira tickets.
   E4 Both: GitHub issues and Jira tickets.
```

Mapping: E1 → `none`; E2 → `github`; E3 → `jira`; E4 → `both`.

### sdd-po new modes

```markdown
## Mode: analyze-ticket (pre-propose enrichment)

Input: ticket_key (string), artifact_store.mode
Output: enriched_context (markdown) with:
  - Completeness score (description: ✓/✗, acceptance_criteria: ✓/✗, scope: ✓/✗)
  - Structured requirements extracted/inferred
  - Gaps identified

## Mode: post-apply-report

Input: change_name, artifact_store.mode
Output: Jira comment update with:
  - Test summary table (pass/fail counts)
  - Test names and statuses
  - Coverage percentage (if available)
  - Files changed summary
```

### Migration guard update (inject.go)

The content-detection `if` block gains one new check:
```go
strings.Contains(prompt, "E. Issue Tracking")
```

The `preflight` string literal appends group E after group D in both EN and ES blocks.

## Testing Strategy

| Layer | What to Test | Approach |
|-------|-------------|----------|
| Unit | `ensurePreservedOpenCodeOrchestratorPreflight` preserves E group | `inject_test.go`: input with old 4-group block → output has 5 groups |
| Unit | `ensurePreservedOpenCodeOrchestratorPreflight` no-ops when E present | `inject_test.go`: input already has E group → output unchanged |
| Unit | Asset wording for group E | `assets_test.go`: check E group text in opencode orchestrator |
| Manual | sdd-po analyze-ticket mode | Run against a sample Jira ticket, verify structured output |
| Manual | sdd-po post-apply-report mode | Run after a real apply, verify Jira comment format |

## Migration / Rollout

The Go migration function `ensurePreservedOpenCodeOrchestratorPreflight` handles users whose orchestrator prompt was preserved from a previous version (without group E). The function detects missing E-group content and replaces the entire preflight migration block. Users with up-to-date prompts (detected by the guard checks) get no changes.

## Open Questions

- [x] All questions resolved during design
