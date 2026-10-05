---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-claude-env-file.md
    summary: Card 1 misses the settings_test.go assertion in TestBuildSettings_DenyToggleMatrix that expects exactly one PreToolUse entry, the Bash stdin rewrite, and fails once the hook is removed
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/03-driver-bare-step.md
    summary: Card 3 states the envelope_path source once and points back from Parking and Repair records, while D3 says each of the three sections says where friction_dir, scratch_dir and next_interrupt_policy are read from
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/02-short-step-envelope.md
    summary: docs/overview.md line on the generic step envelope says it names friction_dir and scratch_dir on success and every error, which the short envelope makes false, and no card corrects it
  - id: F4
    severity: NIT
    class: decision
    location: _lyx/plan/02-short-step-envelope.md
    summary: The last_step.envelope_path test is left as an either/or between extending TestStatusCmd_LastStep and the new table
---

### [MEDIUM] Card 1 misses a settings test that pins the deleted hook

**Location:** `_lyx/plan/01-claude-env-file.md`, `Tests`, the `settings_test.go` bullet.
**Issue:** `internal/shuttleengine/claudeengine/settings_test.go` `TestBuildSettings_DenyToggleMatrix` ends with `if !tt.wantAgentEntry && !tt.wantAskUserEntry && len(preToolUse) != 1 { ... want only the Bash stdin rewrite entry }`.
With the unconditional Bash hook gone, a non-fork row with no Agent and no AskUserQuestion entry has zero PreToolUse entries, so that assertion fails and the plan's `go test` verify line fails with it.
The card names only `TestBuildSettings_BashStdinRewrite` for replacement.
**Fix:** add a bullet to the card's Tests: update the `TestBuildSettings_DenyToggleMatrix` tail assertion, which now expects an empty PreToolUse list when nothing is configured (the webster fork guard is covered by `TestBuildSettings_ForkContextWebsterGuard`).

### [MEDIUM] Card 3 says the envelope_path source once where D3 says it in each section

**Location:** `_lyx/plan/03-driver-bare-step.md`, `Intent`, the `## Parking` and `## Repair records and the stop report` bullet.
**Issue:** D3 says `## Error envelopes`, `## Parking` and `## Repair records and the stop report` each say that `friction_dir`, `scratch_dir` and `next_interrupt_policy` are read from the file at `envelope_path`, or from the printed envelope when it carries none.
The card says it once and has later sections "point back rather than restate", and "the first section that needs them" is ambiguous against the `## Error envelopes` bullet that already says it.
The record lists no licence to say it fewer times, and a driver reads each section when it reaches that stop.
**Fix:** state in the card that each of the three sections says it in one sentence of its own.

### [LOW] docs/overview.md goes stale on the step envelope

**Location:** `_lyx/plan/02-short-step-envelope.md` `Edit` list.
**Issue:** `docs/overview.md` (the shed module paragraph on traces) says the generic `step` envelope names `friction_dir` and `scratch_dir` on success and every error.
After this task stdout carries neither by default.
D4's premise that `docs/overview.md` needs no change rests on no module or execution-stack change, but this sentence describes the observable CLI behavior the task changes, and CLAUDE.md requires changed observable CLI behavior to land with its docs.
**Fix:** add `docs/overview.md` to card 2's `Edit` list with that one sentence corrected to the short envelope and the `envelope_path` record; no module table or execution stack entry changes.

### [NIT] Open choice on the last_step test

**Location:** `_lyx/plan/02-short-step-envelope.md`, `Tests`, the `TestStatusCmd_LastStep` bullet.
**Issue:** "`TestStatusCmd_LastStep`, or the table above" leaves the author to choose.
**Fix:** name one: extend the new table, which already holds the short envelope's `envelope_path`, with a `statusCmd` read over the same `StepsDir`.

## Checked and found clean

- Card 2 `internal/loomcli/cli.go` and `pattern/PATTERN-shed-verb-set.md` are licensed by D2 (the step envelope's contract) and D4 (docs land with the code); the `Long` text states the old key claim, and no test pins it.
- Card 1's deletion of `internal/shuttlecli/smoke_bashstdin_test.go` is licensed by the Scope's "tests that cover the changed behavior"; it holds one test and no shared helper, and `go vet -tags smoke ./internal/shuttlecli/...` still compiles the rest.
- The `--full` record-identical criterion, the `last_step.envelope_path` equality and the pwsh `WithEnv` form each have a carrying card.
- No test outside `parity_test.go` and the `steprecord_test.go` set reads `scratch_dir`, `friction_dir` or `trace_id` from a recorded step's stdout.
- Every targeted package's tests run in the `## verify:` lines.
