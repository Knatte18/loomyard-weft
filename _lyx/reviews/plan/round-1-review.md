---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/01-shuttle-ttl-config.md, _lyx/plan/03-claudeengine-prompt-cache-ttl.md
    summary: cards 1 and 3 each add a top-level test function to a package already at its test budget, and no card raises the budget row, so the untagged gate fails
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/01-shuttle-ttl-config.md:18
    summary: the template.yaml trailing comment names the accepted values 5m and 1h in a shuttleengine file, beyond the template defaults the decision record allows there
---

### [BLOCKING] Cards 1 and 3 exceed the per-package test budget with no budget raise

**Location:** `_lyx/plan/01-shuttle-ttl-config.md` (Tests, `TestLoadConfig_PromptCacheTTLKeys`), `_lyx/plan/03-claudeengine-prompt-cache-ttl.md` (Tests, `TestPrepare_LogsResolvedPromptCacheTTL`)

**Issue:**
`cmd/lyx/testdata/test-budget.yaml` pins the most top-level `func Test…` functions a package may hold, and `cmd/lyx/testbudget_test.go` (untagged, so part of `go test ./...`) fails when a package exceeds its row.
`internal/shuttleengine` holds 140 and its row is 140; `internal/shuttleengine/claudeengine` holds 89 and its row is 89 (counted as `isTopLevelTest` counts: `Test` plus a non-lowercase rune, `TestMain` excluded).
Card 1 adds one new function to `shuttleengine` and card 3 adds one new function to `claudeengine`.
Both new functions are justified under `PATTERN-test-economy` (no existing test covers the open-map load behavior or the log line, and the log test cannot be a row because it must not run in parallel), so the fix is the raise `PATTERN-test-speed` prescribes, not folding.
Neither card lists `cmd/lyx/testdata/test-budget.yaml` among its targets, so each card's commit fails the untagged gate, and so does the plan's `go test ./...` step.
The `internal/webstercli` (81 of 81) and `internal/configreg` (6 of 6) packages gain no function, so their rows stay.

**Fix:**
Add `cmd/lyx/testdata/test-budget.yaml` to the `**Edit:**` targets of card 1 (`internal/shuttleengine` 140 to 141) and card 3 (`internal/shuttleengine/claudeengine` 89 to 90), and say so in each `**Intent:**`.

### [LOW] The shuttle.yaml template comment names the accepted TTL values

**Location:** `_lyx/plan/01-shuttle-ttl-config.md:18` (the `template.yaml` bullet)

**Issue:**
The decision record's constraint and acceptance criterion say no `shuttleengine` file names the `5m|1h` vocabulary beyond the template defaults, checked in review under `PATTERN-shuttle-provider-seam`.
The card has the template's trailing comment end with "accepted values are `5m` and `1h`", which is the vocabulary itself in a `shuttleengine` file and not a default.
The engine's refusal already names the accepted values (D4), so the operator is told where it matters.

**Fix:**
Reword that part of the comment to say the values are Claude Code's prompt-cache TTLs, validated when a run starts, and that the refusal names the accepted ones.

### Verdicts on the focus questions that produced no finding

- **Fidelity.** D1 to D6 and every constraint are carried.
  D1: card 3 `command.go` and `Prepare`.
  D2: card 1, with `ConfigOpenMaps` in card 1 and the registry in card 2.
  D3: `resolvePromptCacheTTL`.
  D4: the whole-set walk in sorted order, with the refusal rows.
  D5: the `logger.Info` line and its test.
  D6: the template map and the `orch` and empty-role rows.
  The docs scope is carried: `shuttleengine` `doc.go` in card 1, `claudeengine` `doc.go` and `docs/overview.md` (the shuttle module entry that already lists `claude_deny_agent_tool`) in card 3.
  No card adds scope the record does not license.
- **Verify coverage.** The section runs in the template's order, and the four targeted packages (`shuttleengine`, `claudeengine`, `configreg`, `webstercli`) are covered by `go test ./...` and by the `-tags integration` run.
  The `tmux` and `llm` tiers are compiled by `go vet`, which the rubric allows.
  The fixtures' other `Prepare` callers (tmux and llm smoke tests in other packages) build their `Config` through `LoadConfig`, so none needs an edit.
- **Test economy.** Apart from the budget, the new tests are justified: the open-map load rows are covered by no existing test, and `TestPrepare_LogsResolvedPromptCacheTTL` is the only log assertion.
  The refusal and routing cases are rows in `TestPrepare_RejectedBeforeArtifacts` and `TestPrepare_ThreadsSpecIntoLaunchCmds`.
- **Attack surface.** Card 3's `**Intent:**` states what the guard blocks (a new run of any role, `ResumeSessionID` adoption included), what leaves a started session alone, and that it adds a guard and lets nothing through.
  The env assignment's override of operator settings, and its one exception `FORCE_PROMPT_CACHING_5M`, are stated in the `command.go` const comment and the `doc.go` paragraph.
- **Granularity.** The card boundaries are sound: card 2 is a different package's registry entry that cannot be bundled into card 1 without crossing packages, and card 3's resolution, builders, `Prepare`, docs and fixtures cannot compile apart because the builder signatures change.
