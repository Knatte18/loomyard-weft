---
round: 2
focus:
  - "tools/retro/retro.sh: check that no `_lym` or `.lym` path is written inline in a git or file command, and that every such path comes from one variable block under `set -euo pipefail`, as PATTERN-lym-files allows for a tools/ script."
  - "internal/retrocli/cli.go parseWindow and internal/retroengine/retro.go Window.end: settle what a --until later than now does, and whether a retro stamped in the future can leave runs landed before that stamp uncovered by the next bare retro."
  - "internal/retrocli/cli.go fileRetro: check that a test reaches the filing and release of the darn task, the non-prime, missing tools/retro and duplicate-slug refusals, the parsed --since and --until window, and the success envelope."
  - "internal/retroengine/retro.go Plan.TaskFields: check that a test pins the recipe, profile, kind, labels, slug and body of the filed task."
  - "tools/retro/figures.jq and tools/retro/transcript.jq: check that the integration test fixture reaches the producers-against-medians outlier rule, the gate-fixer sessions column and the background-task gate-failure path, and that each assertion fails when its computation is broken."
  - "internal/retroengine/retro.go Select: check the cap branch when runs share a landing second across the cut, the named-run From against the stencil's `landed after` sentence, and the lookback refusal wording and test."
  - "_lym/landing/summary.md against the tools/retro test: check that every table the summary says the test checks is read by it."
---

Round 2 re-reviews the fixes to the round 1 findings from the diff, with no mandate beyond the rubric.
