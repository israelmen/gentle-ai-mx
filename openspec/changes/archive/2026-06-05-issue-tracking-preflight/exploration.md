## Exploration: Issue Tracking Preflight Option

### Current State

The SDD workflow has a "Session Preflight" mechanism that collects user preferences before any SDD work begins. Currently it covers 4 groups (A–D): Pace, Artifacts, PRs, and Review.

**Two distinct preflight patterns exist across platforms:**

1. **Compact preflight block (opencode only)**: `internal/assets/opencode/sdd-orchestrator.md` has a structured A/B/C/D prompt with option codes. This is also duplicated in Go code at `internal/components/sdd/inject.go:823` (`ensurePreservedOpenCodeOrchestratorPreflight`) as a migration fallback for older installations.

2. **Separate questions (all other platforms)**: Claude, generic, gemini, codex, cursor, kimi, qwen, kiro, windsurf, antigravity orchestrators ask execution mode, artifact store, and delivery strategy as separate questions during first SDD invocation. They do NOT use a compact A/B/C/D block.

**Config propagation pattern**: Orchestrators cache choices and pass them to sub-agents as named parameters:
- `artifact_store.mode` → every sub-agent
- `delivery_strategy` → `sdd-tasks` and `sdd-apply`
- `chain_strategy` → `sdd-tasks` and `sdd-apply`

**The `sdd-po` skill** (`skills/sdd-po/SKILL.md`) already handles Jira integration. It reads config from `sdd-init/{project}` engram or `openspec/config.yaml` under `jira:` section. If `jira.enabled: false`, the skill is completely skipped.

**The `issue-creation` skill** handles GitHub Issues creation with templates, labels, and approval workflow.

**The `branch-pr` skill** handles PR creation with issue linking (`Closes #N`).

### Affected Areas

- `internal/assets/opencode/sdd-orchestrator.md` — add group E to compact preflight block (English + Spanish), add mapping, add caching/propagation instructions
- `internal/components/sdd/inject.go` — update `ensurePreservedOpenCodeOrchestratorPreflight()` Go function with group E (lines 823–901)
- `internal/assets/claude/sdd-orchestrator.md` — add issue tracking question to the separate-question flow
- `internal/assets/generic/sdd-orchestrator.md` — same (has both model-capable and model-small sections)
- `internal/assets/gemini/sdd-orchestrator.md` — same
- `internal/assets/codex/sdd-orchestrator.md` — same
- `internal/assets/cursor/sdd-orchestrator.md` — same
- `internal/assets/kimi/sdd-orchestrator.md` — same
- `internal/assets/qwen/sdd-orchestrator.md` — same
- `internal/assets/kiro/sdd-orchestrator.md` — same
- `internal/assets/windsurf/sdd-orchestrator.md` — same
- `internal/assets/antigravity/sdd-orchestrator.md` — same
- `internal/assets/skills/_shared/persistence-contract.md` — add `issue_tracking` to the sub-agent context protocol (what gets passed in prompts)
- `internal/assets/assets_test.go` — update preflight tests that validate wording (lines 295–394)
- `internal/components/sdd/inject_test.go` — update tests for the Go migration function

### Approaches

1. **Minimal: Config-only in orchestrators (recommended)**
   - Add group E to opencode's compact preflight and equivalent question to all other orchestrators
   - Cache as `issue_tracking` with canonical values: `github`, `jira`, `both`, `none`
   - Pass `issue_tracking` to sub-agents that need it: `sdd-propose` (to decide whether to trigger sdd-po), `sdd-apply` (to create GitHub issues/PRs), `sdd-archive` (lifecycle updates)
   - The sdd-po skill already handles Jira when activated — no changes to it
   - The issue-creation and branch-pr skills already work standalone — no changes to them
   - Pros: Minimal footprint, no skill rewrites, clean separation of concerns
   - Cons: Orchestrators need to know WHEN to invoke sdd-po based on this config
   - Effort: Medium (11 orchestrator files + Go code + tests)

2. **Extended: Config + orchestrator routing logic**
   - Same as Approach 1, PLUS add explicit routing rules to orchestrators: "If `issue_tracking` includes `jira`, delegate `sdd-po` after `sdd-propose`. If it includes `github`, ensure `issue-creation` skill is loaded for PR phases."
   - Pros: Complete end-to-end flow
   - Cons: More lines changed per orchestrator, higher review burden
   - Effort: Medium-High

3. **Centralized: Add issue_tracking to sdd-init config**
   - Store `issue_tracking` in the `sdd-init/{project}` engram artifact alongside other project config
   - Orchestrators read it from there instead of preflight
   - Pros: Persisted across sessions, single source of truth
   - Cons: Breaks the "preflight is per-session" pattern; issue tracking preference may vary per session

### Recommendation

**Approach 1 (Minimal: Config-only)** is recommended. Here's why:

- It follows the EXACT same pattern as the existing 4 groups
- It doesn't touch any skill files (issue-creation, branch-pr, sdd-po)
- The orchestrator already knows how to route to sdd-po (it's a skill); we just need the config to decide WHEN
- Keep orchestrator routing logic simple: "if `issue_tracking` includes `jira`, include sdd-po in the skill resolution for propose/archive phases"

**Propagation flow:**
```
Preflight → user picks E1/E2/E3/E4
         → orchestrator caches `issue_tracking: github|jira|both|none`
         → orchestrator passes `issue_tracking` to relevant sub-agents
         → sdd-propose: if jira, also delegate sdd-po
         → sdd-apply/archive: if jira, delegate sdd-po for lifecycle updates
         → branch-pr: if github, ensure issue linking (already default behavior)
```

### Risks

- **Jira MCP unavailable**: If user picks E2/E3 but Jira MCP server is not configured, sdd-po will fail. Mitigation: sdd-po already handles this gracefully (backend detection protocol). The orchestrator should warn if Jira skill is not in the registry.
- **Test updates**: The Go test file `assets_test.go` has specific preflight wording assertions (lines 295–394) that will need updating. Missing these will cause CI failures.
- **Migration function**: The `ensurePreservedOpenCodeOrchestratorPreflight` function in `inject.go` has a complex validation check (line 904–912) that determines if the preflight is "current enough". Adding group E means updating BOTH the template AND the validation conditions.
- **11 orchestrator files**: All platform orchestrators need the same logical change with platform-specific phrasing. Risk of inconsistency. Mitigation: use the generic orchestrator as template, adapt per platform.
- **Spanish localization**: The opencode preflight has a Spanish translation that must stay in sync.

### Ready for Proposal

Yes — the scope is well-defined: add a 5th preflight group across all orchestrators, update the Go migration function, and update tests. No skill files need modification. The orchestrator routing for sdd-po activation can be kept minimal (just pass the config; sdd-po already has its own activation logic via `jira.enabled`).
