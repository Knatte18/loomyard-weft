# Fixer report — webster round 1

## Changes

- F1 (fixed as proposed), `5aba9de6`: `MaxEntryLineChars` in `internal/pattern/check.go` is 360.
  `internal/pattern/doc.go` names the constant, not the number, and no test pins the literal, so nothing else changed.
  `lyx gate test ./internal/pattern` passes, including `TestOverview_RealFilePassesCheck`.
- F2 (fixed as proposed, then superseded by the headroom gate), `efdb9da9`: the eleven listed entries in `PATTERN.md` carried the parent's text again, which made the overview 16091 bytes.
  The verify gate's headroom check then refused: the overview grew past the 15564-byte headroom line from the parent's 15736 bytes, and the declared `PATTERN-sparse-hub-base` line alone adds 355 bytes.
  The review's fallback has no bytes to give: the prose outside the entries is a title and one sentence, and the declared entry's text is fixed.
  The gate-fix commit trims the same eleven clauses again, so the overview is 15719 bytes, under the parent's size.
  Each trimmed clause is held by its entry's background file, checked by grep.
  The eleven entries therefore differ from the parent's overview text; the gate's headroom rule and the review's restore rule cannot both hold, and the gate decides.
- F3 (fixed as proposed), `ff94a079`: `TestRun_PassesTheEnvironment` also calls `RunWithEnv` with an `extraEnv` entry that differs from the `t.Setenv` value and asserts the child prints the `extraEnv` value.
  `lyx gate test --tags integration ./internal/testkit/lyxbin` passes.
- F4 (fixed differently), `8cb18896`: the review's premise is that the tmux-tagged helper file is not compiled in the integration tier, and the proposal is a new shared variable in an untagged file.
  `smoke_helpers_test.go` is tagged `integration || tmux`, so the integration file already reaches `skipWalkPush`.
  The fix deletes the local `skipPush` in `smoke_prebootstrap_integration_test.go` and points its two callers at `skipWalkPush`, with no new variable and no move.
  `lyx gate test --tags integration ./internal/loomcli` passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, MEDIUM: the restored overview, 16091 bytes, exceeds both the parent's 15736 and `HeadroomBytes`, so the headroom check might trip. Disputed, premise false: `lyx loom lint-pattern`, a caller of the `patternlint.Lint` that the verify gate calls, returns ok on this tree.
- fix-proposal, LOW: the 360 cap leaves 40 characters of regrowth room for the declared entry. Disputed, no change: it is the reviewer's own proposed number and breaks nothing, and the rubric excludes a further improvement the review never asked for.

## Disputed

- fix-proposal MEDIUM (headroom): premise false, evidence is the passing `lyx loom lint-pattern` run above.
- fix-proposal LOW (cap slack): not a finding under the fix-check rubric; the 360 value is the review's own.

## Deferred

None.
