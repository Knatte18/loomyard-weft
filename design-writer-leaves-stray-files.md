# A writer session can end with untracked files in the worktree, which halt Finalize

`writer-leaves-stray-files` · Task · bug, shuttle, darn, loom · running · Code-Burler-Review

## Seen (orch, 2026-11-01)

The darn writer of `batch-budget-300k-trial` left two untracked files at its worktree root, `200k` and `250k`, each one stray line ("cautious base est", "wide est"), most likely from a shell redirect in a command it ran.
Its session ended and the run went on through review to Finalize, which refused the merge-in over the uncommitted files; the tackler handed the stop back and the orch removed them by hand.
Again the same day, the Parent-Align fixer of `reed-status-buttons` wrote its report with verdict `fixed` but left `.scratch-task.txt`, a list of the task's changed paths, untracked at the worktree root; the run halted at Parent-Align with "the fixer session ended without a usable turn (dirty)", and the orch removed the file by hand and resumed.
Such files should never exist at the end of a writing session.

## Idea

A producing session (darn writer, Webster fork, fixers) cannot end while its worktree holds untracked or uncommitted files it did not commit: its end gate, beside the verify and description gates shuttle already runs, names the files and tells the agent to commit what belongs to the change and delete the rest, so the stop is fixed by the agent that made it, not at Finalize.
