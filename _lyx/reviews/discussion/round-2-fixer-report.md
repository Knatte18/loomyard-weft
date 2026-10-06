# Round 2 fixer report

All eleven findings are fixed in `_lyx/discussion/decision-record.md`; the question ledger of `_lyx/discussion/support-log.md` gained Q12.

## Changed

- F1 (D7): the refusal for a remote holding commits the local HEAD lacks now names a plain merge and, on the operator's judgment that the listed commits are the run's own abandoned ones, `git merge --strategy ours`; the Rationale no longer implies the guard covers the #404 state without a merge; the acceptance criterion names the #404 state converging after the merge.
- F2 (D7): the remote update moves inside the pair-warp reset, after its gate checks and before the checkout rewrite, so a local refusal changes nothing; a non-lease remote failure refuses with a re-run way forward; a failed checkout rewrite after the remote moved states that the remote was updated.
- F3 (D6): the Rationale now says the batch is terminal and `recover-batch` stops any live prior strand, with the dead batch's kept strand named.
- F4 (D4): the `recover-batch` check runs after its uncheckable-paths refusal, so D4's way forward is reached only by a batch `recover-batch` accepts.
- F5 (D3): the bound names the real one (the Plan-Gate saw the target exist) and says `delete-not-done` does not bound the residue.
- F6 (D1 to D4): one scope phrase, unbegun cards, with D4 stating the deleting side and the later cards it compares.
- F7: the D6 acceptance criterion names `dead`.
- F8 (D7): the update is skipped when the remote tip is an ancestor of the reset commit, and the acceptance criterion covers it.
- F9 (D1): the card-number comparison and its same-batch consequence under a grouping batchifier are stated.
- F10 (D10): the Rationale is corrected and a Bound states what the removal lets through.
- F11: the duplicate Open risks bullet is deleted.

## Deferred

Nothing deferred.
