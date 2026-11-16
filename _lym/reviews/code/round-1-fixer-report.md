# Round 1 fixer-report

Every finding is fixed as proposed unless marked otherwise.
The gate (`lym gate test --tier 0 --tier 1` over `internal/retroengine`, `internal/retrocli` and `tools/retro`) is green.
Commits group the findings by the files they share: `tools/retro`, `retroengine`, `retrocli`, and one follow-up for the fork findings.

## Changes

- **F1:** `tools/retro/retro.sh` declares `USAGE_DIR`, `SHED_DIR` and `REVIEWS_DIR` below `set -euo pipefail`, and the ls-tree and show calls use them.
- **F2:** `parseWindow` in `internal/retrocli/cli.go` refuses an `--until` later than now, naming the flag, the value and the way forward. A refusal row is in `TestRetro_RefusesBadArguments`.
- **F3:** new `internal/retrocli/fileretro_integration_test.go` drives `lym retro` through `clihelp.ExecuteIn` over a hubforge hub with one landed run.
  It covers the non-prime and missing-`tools/retro` refusals, the filed and released task (null status, darn recipe, docs profile, research label, body), the envelope, the duplicate-slug refusal, and the last retro read from the board and from a report directory.
  The hub's `board.yaml` is given the `research` type, which the fixture's board vocabulary lacks.
  `testmain_test.go` now sets the hermetic git environment.
- **F4:** `TestPlan_TaskFieldsFilesOneDarnTaskOnTheDocsProfile` pins the task's slug, title, kind, recipe, profile, labels and body.
- **F5, F11:** the `tools/retro` fixture holds three main sessions of one row with one outlier, and a `gate-fixer` session. The test asserts the medians line and the Runs row's gate-fixer cell.
- **F6:** `Select` leaves out every run landed in the first left-out run's second.
  Fixed differently from the proposal's "keep at least one" (see the self-check): when the cap falls inside the oldest landing's second, it keeps that whole second past the cap.
  Rows cover both cases.
- **F7:** the duplicate-slug refusal names a way forward for each kind of stamp.
- **F8:** named runs start a second before the oldest landing, and the `TestSelect` named row expects it.
- **F9:** the success path of `parseWindow` is pinned by the F3 test's `--since`/`--until` refusal, which names both parsed ends.
- **F10:** the transcript fixture has a gate call moved to the background, failed only by a task notification. `GateRuns` and `GateFails` are 2.
- **F12:** the named-run refusal names the 60-day lookback and its start, and gives `--since` and `--until` as the way to an older run. It also covers a misspelt slug. A `TestSelect` row asserts the lookback date.
- **F13:** the test now asserts `facts.jsonl` and `usage.jsonl`, so the claim in `_lym/landing/summary.md` holds. That file is not edited, since it lies under `_lym`.
- **F14:** `retro.sh` names the transcript layout as Claude Code's in its header and records the missing directory as `transcript directory (Claude Code layout)`. The test expects it.

## Gate findings repair

The comment gate found three comments wrapped mid-sentence:
the header and `seedLandedRun` comments of `internal/retrocli/fileretro_integration_test.go`, and the cap comment in `Select`.
Each is now one sentence per line, under 120 characters, and the tiers 0 and 1 gate is green again.

## Fixed differently from the proposal

- **F6:** the proposal's "keeping at least one" still lost the tied runs when every kept run shared the left-out run's second (fork finding F-A).
  The change keeps the whole tied second instead, which can exceed the cap.

## Disputed

None.

## Deferred

None.

## Rulings

None reached me.

## Self-check

- fix-proposal, MEDIUM, F-A: the cap tie reaching the first kept run widened the loss. Fixed with my own change (above), with a `TestSelect` row.
- fix-proposal, LOW, F-B: the lookback refusal blamed the lookback for a misspelt slug. Fixed: the text covers both causes.
- fix-code-correctness, LOW: a `retro.sh` header line passed 120 characters. Fixed: split into two lines.
- fix-consistency: no findings.
- fix-affected-set: no findings.

Each fork's report is saved as `round-1-fixer-report.lens-<lens>.md` beside this file.
