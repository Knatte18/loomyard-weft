# Move rules about lyx behaviour out of this repo's CLAUDE.md into lyx code, doc.go and stencils

`claude-md-lyx-rules-out` · Note · enhancement, stencil, board, loom

## Seen

This repo's `CLAUDE.md` holds rules about how lyx itself behaves, which hold for every repo developed with lyx, not only this one.
Examples: in "Docs land in the same commit", that the task that builds a design never changes its own board entry's body or scope, that the landing marks the entry done and the board prune removes it, and that no decision, card or step of a run changes the running task's own entry; in "Production", how Batten's Land row runs the hub's `deploy` command and how `alignment_pr_approve` withdraws the alignment agent's landing.

## Idea

`CLAUDE.md` is for one repo only (operator, 2026-10-22).
Go through it and move every rule about lyx's behaviour out: what lyx enforces goes into the code and its package `doc.go`, and what agents must be told goes into the stencils lyx ships, so it reaches every hub.
What stays in `CLAUDE.md` is what holds for this repo alone, such as the cgo build, `update-plugins.sh` and the push rule for `main`.

The run-lock rule is handled by note `entry-align-before-discussion`.
