---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Fixer report, code round 1

## Changes

- F1: removed the golden paragraph from `internal/shellguard/doc.go`, and the sentence in `_lym/landing/summary.md` that said the package doc names the `-update` command.
- F2: declared `settingsGoldenPath` once in `internal/shuttleengine/claudeengine/settings_test.go`; the flag's usage string and the golden's read and write use it.
- F3: added `unquotedWordBreakers` and `unquotedWordRunEnd()` in `internal/shellguard/rows.go`; the word unit and the run's end are built from that one set, and `sessionBusCommandPattern` uses the function.
  Rewrote the `unquotedWordOtherThan` doc, the `doc.go` sentences and the matching summary text.
  The end bracket now lists the same characters in a new order, so `testdata/settings-golden.json` was regenerated with `-update`.
- F4: the Merge danger blast radius in `_lym/landing/summary.md` names shellguard and the claudeengine settings golden test.

Commits:

- `4a83092c6`: F1, F3, F4
- `fcbccde0d`: F2
- `b9d92e1d7`: the self-check's correction to the F3 doc comment
- `925e67e73`: the gate's line-length finding on the `unquotedWordBreakers` comment, split into two sentences

Verification:

- `lym gate test` on `shellguard`, `claudeengine` and `codexengine`, at tier 0 and `--tier 1`: no failures.
- `TestBuildSettings_MatchesGolden` ran under `-update` at tier 0, and the compare run followed in the tier 0 pass.
- `TestRows_PatternsDenyAndAllow` ran and passed at tier 1.
- After each of `b9d92e1d7` and `925e67e73`, both comment-only changes, `shellguard` was rerun at tier 0: no failures.

## Self-check

- fix-code-correctness, MEDIUM, `unquotedWordOtherThan` doc overstates what the end matches: fixed as proposed (`b9d92e1d7`).
- fix-consistency, MEDIUM, the same sentence: fixed as proposed (`b9d92e1d7`).
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
