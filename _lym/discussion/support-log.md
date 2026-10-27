## Interview

A darn run holds no interview.

## Rejected alternatives

- Running the three handlers one after another: rejected, the entry counts the medium runs as parallel and a sequence would only lengthen the bench.
- Rewriting the harness around a shared scratch directory: rejected, the earlier benches' scripts take one root and a per-run cell, so changing the root is the smaller change.
- Adding a fourth effort level or a Claude run for comparison: rejected, the entry's cap forbids both.

## Review rounds

_No rounds yet._

## Question ledger

- Where do the harness scripts run? Self-pick: from this worktree's `.scratch/`, which `.gitignore` excludes.
- Which date names the output directory? Self-pick: the day the runs finish.
- Does a copy of the input per run suffice for parallel handlers? Self-pick: yes, `run-forkfan.sh` already copies `work` into a per-cell directory; the cell name gains the run label.
