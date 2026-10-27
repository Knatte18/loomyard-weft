## Interview

A darn run holds no interview; the board entry is the whole statement of the task.

## Rejected alternatives

- Running the three cells one after another in a shared `CODEX_HOME` with a shared working tree: rejected, a later run could read an earlier review file.
- Passing a model or effort to `spawn_agent` for the forks: rejected, the earlier benches let forks inherit the handler's own, and the board entry asks for identical settings but for effort.
- Regenerating the prompt from the block's wording by hand: rejected, the earlier build script inserts the block as one line into the boilerplate, and reuse keeps the conditions equal.

## Review rounds

_No rounds yet._

## Question ledger

- Where do the harness scripts point their path root? Resolved: at this worktree, `codex-v2-p4-bench`, self-picked.
- Which date names the output directory? Resolved: the date of the run, from `date +%F` when the report is written, self-picked.
- Does the stop at 70 percent apply to the five-hour window as well as the weekly one? Resolved: to either, self-picked.
