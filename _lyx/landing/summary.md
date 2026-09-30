# Webster: survive a parent merge between a batch commit and record-batch

A parent merge-in that landed after a fork committed but before `record-batch` (or `recover-batch`) wedged the run.
The report's `head_sha` no longer equalled the worktree HEAD, so `record-batch` refused, and `begin-batch`/`recover-batch` refused because an OK report already existed.
The only way out was removing the merge commit by hand.

## What changes

- `record-batch` and `recover-batch` share one head-reconciliation rule in `internal/websterengine`.
  HEAD is accepted when it equals the report's `head_sha`, or when HEAD's first-parent chain reaches `head_sha` through merge commits only.
  A non-merge commit on that walk, or a walk that never reaches `head_sha`, is refused with an error naming both SHAs and the merge-only rule.
- `record-batch` records the batch at the report's own `head_sha` for `CardSHAs`, the post-batch delta and drift's triggering SHA, so the parent's changes are not attributed to the batch's cards.
  `DoneChecks` and drift's tree resolution still read the merged worktree, since the rest of the plan builds on that tree.
- An accepted merge adds one warning to the result naming the report's `head_sha`, the current HEAD and each walked merge SHA.
- Both verbs refuse while a git merge is in progress, pointing at `lyx fabric merge --continue`/`--abort` (hub) or `git merge --continue`/`--abort` (standalone).
  The probe asks git whether `MERGE_HEAD` resolves, so it works in linked worktrees.
- `websterengine` exports an in-flight predicate: `state.json` exists and `outcome.yaml` is absent or not `done`.
  `paused` and `stuck` count as in flight.
- `lyx fabric merge-in` warns (never refuses) when a Webster run is in flight, on a merge that moved HEAD and on the conflict envelope, under a `warnings` array.
  A malformed `outcome.yaml` degrades to no warning plus a log line.
  `fabricengine.MergeIn` is untouched, so landing's own merge-in is unaffected.
- `ly-drive` gains a rule to prefer not merging the parent mid-Webster, and that `merge-in` is survivable and warns.
- Docs: `websterengine` package doc and `merge-in`'s `Long` help.

Behaviour when HEAD equals `head_sha` is unchanged.

## Design notes

- The walk and the merge probe are read-only git queries, so the Fabric Git Invariant holds.
- The predicate takes the anchor root it is handed and builds paths only through webster's own helpers, keeping the Cwd Resolution Invariant.
- Known limits, unchanged by this change:
  - A merge-in that fast-forwards (the fork's commit is an ancestor of the parent tip and the warp has nothing the parent lacks) moves HEAD by non-merge commits and is still refused; recover by resetting the warp to `head_sha` before `record-batch`.
  - Integration bisect walks `CardSHAs` across the merge, so a card recorded before it is verified on a tree without the parent's changes; the merge-in warning says so.
  - A fork that runs `git merge` after writing its report is accepted; the warning naming the merge SHAs is the only trace.
  - Symbols the parent renamed or deleted fall in no batch's delta, so drift does not repair them; the next `begin-batch` dispatch validation reports them.
  - Standalone runs never see the `merge-in` warning, since `lyx fabric` is hub-only.

## To check by hand

- Run a real hub loop, merge the parent in mid-Webster with `lyx fabric merge-in`, and confirm the warning and a clean `record-batch` afterwards.
- Conclude a conflicted merge-in with `lyx fabric merge --continue` and confirm `record-batch` then succeeds.
