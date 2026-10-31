# README rewritten as a teaching text, overview figure brought up to date

`README.md` is rewritten from scratch for a reader new to LoomYard, and `docs/loomyard-overview.svg` is corrected against the code.
The previous README is kept byte-identical as `README.old.md` for comparison.

## What changed

- `README.md` covers, in order: what LoomYard is and the problem it solves, hub and worktrees, the board and its tasks, runs, beams with dobby and orch, a task's path from the board to a landed commit, getting started, and links into `docs/`.
  It has no module table; weaving terms appear only where the path needs them, glossed there.
  It embeds the figure once, at the task's path.
- `docs/loomyard-overview.svg` now has dobby start the batten run, where the orch was drawn starting it.
  The loom row shows the current rows (Discussion, Plan, Webster, Parent-Align, Code review, Describe, PR-Review, Finalize), with the retired Publish, PR-Align and PR-Gate boxes gone and the review segments bracketed separately.
  The glossary gains dobby and beam in place of two entries the figure never used.
- `docs/overview.md`'s sentence about the figure names the dobby start.
- Corrections to the old README's claims: the git floor is read from `internal/toolfloor` rather than pinned, and landing is described through Describe, PR-Review and Finalize.

## Design decisions

- The records side is written `<records>` in the layout sketch, since `TestEnforcement_FabricVocabulary` allows no bare side token in `README.md` beyond the public repository's name.
- Commands in the README were read from `lym help` and the recipes under `contracts/recipes/`; the getting-started steps were not run on a fresh machine.
- The figure was edited as SVG text and checked with `xmllint`; it was not rendered in a browser.
