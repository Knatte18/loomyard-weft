---
round: 1
focus:
  - "internal/stencilstore/engineblock.go and engineblock_test.go: does the one resolver read `engine/<engine>/<block>.md` at call time, and does a missing block name the block, the engine and the file in its error? Is every other reader of stencil sets routed through it, with no second resolver left?"
  - "internal/seatengine (table.go, prompt.go, engine.go, resume.go): is each seat's engine, else the told default engine, passed at every place a seat's stencil set is read, the marker check before any seat starts included? Does a block missing for the chair's engine refuse the table naming block, engine and file? (The assistant-not-starts path belongs to a later part of the project; check only that the diff does not pre-empt it.)"
  - "internal/burlerengine (prompt.go, engine.go, fix.go, review.go) and internal/burlercli/wiring.go, internal/loomcli/wiring.go, darnwiring.go: does composition of each half's instruction files use that half's engine, and does Shuttle's default engine reach seatengine and burlerengine through the wiring? Does a Claude seat's prompt stay byte-identical to before? Check the prompt.go rewrite (about 100 changed lines) for dropped or reordered rules that the decision record did not ask to change."
  - "internal/heddleengine (render.go, target.go): does `codex-file` open with the anchor's AGENTS.md, else CLAUDE.md, else nothing, and does `target.go` re-render it on a CLAUDE.md change? Does it carry every repository-layer entry aimed at `claude-file`, while shipped entries stay on `claude-file` alone? Does a render whose target list names `claude-file` also render `codex-file`, for both a new and an existing worktree? Look for layer-ordering or duplication errors and for a case where no file is rendered."
  - "Scope against D11 and D12 only: the diff touches many test files in darnengine, driftengine, landingshed, loomengine and shedrecipe. Is each such change a signature follow-through of the engine parameter, or does it reach into the other decisions (D1 to D10, D13)? Does any change license assistant seats, codexengine, fslink or config keys?"
  - "PATTERN.md and the decision record's `## PATTERN changes`: does `PATTERN-engine-blocks` land with the declared text (D11)? Is `PATTERN-fs-links`, which belongs to D13, left out of this part?"
  - "Test economy: contracts/stencils/registry_test.go, template_test.go, prompt_test.go, table_test.go, render_test.go and engineblock_test.go. Does any new test duplicate a covering test, test a helper the public surface already covers, or assert static stencil prose? Does any added behavior lack coverage? Is the acceptance criterion on engine block resolution and on the `codex-file` fallback covered once?"
  - "Commit messages and the change description: do the claims in the three commit messages (CLAUDE.md fallback, claude-file carry-over, engine-keyed reads, PATTERN-engine-blocks) match the diff? Is there any Pattern-Departure trailer, and is it justified?"
---

Round 1 seed.
The decision record's Goal is a board entry for part 3 of `codex-engine`, bounded to D11 and D12 of the project Sketch.
The amended Sketch was read through `lym board get P2 --record`.
No `_lym/landing/summary.md` or plan was inspected in this pass.
The focus entries only point at where to look and do not pre-state any verdict.
