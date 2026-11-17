# Round 2 fixer-report

## Changes

- F1 (BLOCKING), fixed as proposed, commit `85daf159`: `internal/retroengine/retro.go` `Select` widening loop is bounded with `cut < len(runs)`; `retro_test.go` `TestSelect` gains the row "a cap inside a second that holds every run keeps them all".
- F2 (BLOCKING), fixed as proposed, commit `4d57395e`: `tools/retro/retro_integration_test.go` `buildRecords` builds every fixture path from `lymdirs.LymDirName`.
- F3 (LOW), fixed as proposed, commit `4d57395e`: s1 costs $0.6 and s3 $1.2, with the usage rows, sessions rows, Runs cost cell ($2.3) and producers line updated.
- F4 (LOW), fixed as proposed, not committed: `_lym/landing/summary.md` Tests section extended for `internal/retrocli` and `internal/retroengine`.
  The file sits under `_lym`, so no git command was run against it.
- F5 (LOW), fixed as proposed, commit `61eff0b0`: `contracts/stencils/retro/retro-template-task.md` line 5 now reads "Write one report and its line in the research index, and change nothing else".

Gate: `lym gate test ./internal/retroengine ./tools/retro` tier0 passed; `lym gate test --tier 1 ./tools/retro` passed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
