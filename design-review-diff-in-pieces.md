# Stage the reviewer diff in pieces that fit one Read

`review-diff-in-pieces` · Task · enhancement, burler, stencil · running · Describe

## Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 1.
In `test-only-functions-in-prod`, the Code-Review reviewer (6b616569, turns 18 to 35) wrote the branch's production diff of about 3,600 lines to `.scratch/prod.diff` itself, read it in five spans, and had its last Read refused for size, so it retried with `allow_large`.
The session cost $5.40 against a row median of $1.63, with the whole diff in its context.

## Idea

Burler writes the round's diff as one file per package, each under the Read cap, and names them in the explore step (`internal/burlerengine/prompt.go`, `contracts/stencils/burler/burler-step-1-explore.md`).
One session shows this; a second oversized review would confirm it.
