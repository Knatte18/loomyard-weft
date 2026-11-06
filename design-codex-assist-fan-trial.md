# Try the Codex assistant reviewer with its lens fan

`codex-assist-fan-trial` · Task · research, codex, shuttle · running · Drift-Verify

## Seen

The assistant reviewer ran solo on Codex in one real darn run, and note `codex-assist-review-trial` holds the result.
The fan, `code_assist_reviewer_fan`, has never run: `contracts/stencils/engine/codex/assist-fan.md` has Codex spawn one subagent per lens with `spawn_agent` inside its one turn, and no lym fork audit covers them.

## Wanted

Find out whether Astra with lenses works, in two parts, and report what happened in `docs/research/codex/review/assist-fan-trial-2026-11-06/report.md`.

### 1. The writer spawns Astra with the fan on real code

Run exactly one `lym shuttle run --model "astra[medium]"` from this worktree, with one retry only after a launch fault that has nothing to do with the fan.
It reviews the landed commit `b93a0a52`, read through `git show`, and changes no source.
Build its prompt from the stencils an assistant reads, `burler-template-assist-reviewer`, `burler-assist-cluster-fan` and the Codex `assist-reading` and `assist-fan` blocks, with the lenses of the `review-code` fan in `lenses.yaml`.
`internal/burlerengine/assist.go` shows how they fit together.

Report:

- Whether a subagent started for each lens, whether every one returned, and whether the assistant waited for them all inside its turn.
- Whether the report parses as a review, by the check `ReviewGateEntry` names.
- The wall time, and the tokens where Codex's records give them.
- What it found, against `round-1-review.md` and the assistant's solo report of that run, at `_lym/reviews/code/` in weft commit `327f541d1d`.
  That review read the code before its fixes, so the comparison is rough, and the report says so.

### 2. The real seat

This task's `run_settings` turn on Code-Review with the assistant fanned, so the review of this report is the real composed seat on a small diff.
The writer does nothing for this part and names it in the report as still to come.

## Bounds

- At most the one run and one retry above, and no headless launch.
- No code change: if the fan fails, the report states what failed and where, and the fix becomes a note.
