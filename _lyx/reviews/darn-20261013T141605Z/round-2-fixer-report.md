# Fixer-report: darn round 2

## Changes

- F1: rewrote `_lyx/landing/summary.md` to describe the committed inline layout.
  The pack section is filled into the `kickstart_pack` marker at the end of `webster-body-implementer.md`, after the batch-report section, and opens with the one-turn read line.
  Dropped the pack file, `## Start here` pointer, `webster-fragment-pack-read` stencil, pack path parameter and the "own file" and "by reference" title wording.
  Kept the FRESH-READ pointer, the `kickstart_pack=empty` pack-off case, the byte-identical no-pack prompt and the recovery prompt taking no pack.
  Not committed: the file is under `_lyx`, which this round must not run git against.
- F2: moved the `promptPath` computation in `BeginBatch` back after the `os.MkdirAll(deps.Geom.PromptsDir, 0o755)` block, before the prompt write.
  `beginbatch.go` has no diff hunk there now.
  `lyx gate test ./internal/websterengine` is green.
  Committed as `websterengine: fix F2`.

## Disputed

None.

## Deferred

None.
