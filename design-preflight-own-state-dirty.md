# Preflight worktree-clean halts on lym's own _lym state files

`preflight-own-state-dirty` · Task · bug, preflight, loom · running · Drift-Await

Imported from [issue #33](https://github.com/Knatte18/loomyard/issues/33).

What happened: `lym loom` halted twice as `blocked` at Preflight with `worktree-clean: uncommitted state changes under _lym`. The dirty paths were lym's own: a deleted `_lym/loom/friction/loom-halt.md` (the note of the previous halt, consumed by reflection) and an untracked `_lym/shed/<slug>/go-settle.jsonl`. No task code was dirty.

Why abnormal: the precondition is tripped by files lym itself wrote or removed between steps, so each halt note or settle log can cause the next halt. The run needs an orch or operator `lym fabric commit` before it can continue (escalation-to-human anomaly).

What to do: have the worktree-clean check ignore, or auto-commit, lym-owned artifacts (friction notes, `_lym/shed` settle logs), or have the step that removes or writes them commit the change.

Provenance: task `commonlib-fluid-report-selection`; notes loom-halt.md, loom-halt-2.md.
