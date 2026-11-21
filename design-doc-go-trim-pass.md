# Trim every package doc.go of what is superfluous

`doc-go-trim-pass` · Task · enhancement, docs · running · Drift-Verify

## What was measured (2026-11-18, `comment-trim-second-pass`)

Production code has 103,762 comment lines against 268,614 code lines, a 27.9 % share.
The package `doc.go` files hold 27,301 of those comment lines; the comment rewrite last week did not shrink them.
Largest: `fabricengine` 2,474, `websterengine` 1,284, `gatecli` 1,273, `reedengine` 1,174, `shuttleengine` 1,075.
The full measurement is in the change description of that run, records commit `e1d2be8076` in `loomyard-weft`.

## Operator decision (2026-11-18)

Cut everything superfluous in every `doc.go`, not in a chosen set of packages.
Superfluous: text that restates the function or type docs, narrates how the code works step by step, repeats itself, or records history.
What stays: a short package overview, the package's contracts and invariants, and a "why" where the code alone does not show it.

## How the work is done (operator, 2026-11-18)

The writer fans the work out over many Agent-tool subagents on Sonnet at medium effort, each given a fixed small number of `doc.go` files,
so no subagent's context grows large.
The writer sets the batch size by file length, splits the largest files (`fabricengine`, `websterengine`, `gatecli`) into a batch of their own,
and checks each subagent's diff before committing.
No code changes; comments in other files are out of scope.
