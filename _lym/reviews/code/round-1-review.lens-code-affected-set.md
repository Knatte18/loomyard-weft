```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens for the batching-nudge diff. I found no finding.

**Set per change (covered or not)**

- **Shipped rule text** (`lym-batching`, `lym-plan-edits-before-gate`): the readers are `internal/heddleengine/store_test.go`, which pins the batching text and is updated, and `internal/heddlecli/verbs.go`, which only uses the slug in examples. Covered.
  - `docs/research/turn-audit-2026-10-28/report.md` quotes the old wording. It is a measurement report, so it is meant to go stale.
  - The `lym-plan-edits-before-gate` text has no test pin.
  - `batchingnudge.go` quotes rule fragments in its nudge lines, and they match the new shipped wording.
- **New `claude.yaml` key `batching_nudge`**:
  - `ConfigSpec` classifies it as `CategoryNone` in `config.go`.
  - The template carries every Design key at its default (150000, 3, 2, 2000, 5).
  - `config_test.go` pins the template defaults.
  - `shuttleengines.go` adds the validator to the module's load check, and `shuttleengines_test.go` pins its refusal.
  - `doc.go` names the keys.
  - `keepwarm_test.go`, `prepare_test.go` and `command.go` only mention the other keep-warm keys, so nothing there is stale.
  - Covered.
- **New hook in the settings document**:
  - `settings.go` adds the PostToolUse entry only when `BatchingNudge.Enabled`.
  - The existing `settings_test.go` PostToolUse assertions assume a zero `Config`, so the hook is off there and they stay valid.
  - The `settings.go` file and type comments are updated.
  - `TestPrepare_BatchingNudgeHook` pins the on and off cases.
  - Covered.
- **New hidden verb `hook-batching`**:
  - `cli.go` registers it, skips the geometry resolution for it, and updates its comments.
  - It carries the `internal` audience annotation and a `Short`, which the `cmd/lym` tree walk checks.
  - `docs/overview.md` has the nudge prose, and the verb list omits the hidden verb.
  - `cli_test.go` has no registration case for it. That is a test-coverage question for the tests lens.
  - Covered.
- **New capability `BatchingNudger`**:
  - `engine.go` declares it.
  - `shuttleengines.go` adds the accessor, `doc.go` describes it, and `TestBatchingNudger` pins it.
  - `internal/shuttleengine/doc.go` lists only record-reading capabilities, not `ToolBoundaryContextRenderer`, so the omission matches the existing pattern.
  - The seam rule holds: provider specifics live under `claudeengine`, and only `shuttleengines` imports it.
  - Covered.
- **PATTERN entries and declared changes**: the diff changes no `PATTERN.md` or `pattern/` file. The directive declares no PATTERN changes, so nothing is owed.
- **Commit messages and Pattern-Departure trailers**: the three commit messages match what their diffs do, and none carries a `Pattern-Departure:` trailer. No difference from the Design needs one.

**Tool calls (6 allowed, 5 used)**
1. `grep -rln keep_warm_cap_roles`: to find every file that lists `claude.yaml` keys, which my context lacked.
2. `grep` for the old shipped rule text and the `lym-batching` slug: to find other readers of the changed wording.
3. `grep` for PostToolUse test assertions and `hook-batching` / `await-forks` mentions: to find tests and docs that assert settings shape or list shuttle verbs.
4. `grep` in `prepare_test.go`, `command.go`, `keepwarm_test.go` and `heddlecli/verbs.go`, plus `git ls-files CLAUDE.local.md` and `ToolBoundaryRenderer` references: to check whether those listings need the new key and whether `CLAUDE.local.md` is tracked. It is not tracked.
5. `grep` for capability listings in `internal/shuttleengine/*.go` and `docs`: to check whether a capability list elsewhere needed the new `BatchingNudger`.
