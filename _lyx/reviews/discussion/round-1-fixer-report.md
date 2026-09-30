# Fixer report — Discussion-Review round 1

All seven findings fixed; `lyx loom validate-discussion` exits 0 on the edited pair.

## Changes

- **F1** (resume-once, unbounded wait): the resume Decision now states that the marker is written only after `Spawn` succeeds and that a `Spawn` error stays the hard error, retried on resume.
  It also states that a Call on a consumed approval says so in its Stuck reason and names re-running `lyx loom approve` as the recovery.
  Two acceptance criteria were added or extended to match.
- **F2** (archive-tag abort point): the archive Decision now places `Remove`'s tag push after its refusals and before its first mutation, so a failed push leaves the whole pair in place.
  It states that the archive runs whatever `remote` says, that an unreachable origin aborts, and that `force` does not bypass it.
  Added an auto-mode assumption for the `force` pick, an Open risk "Offline pair removal", the matching acceptance-criterion clauses, two support-log rejected alternatives, and a ledger entry.
- **F3** (done-wait budget): the done-wait Stuck is now exempt.
  The budget Decision's flag rule and rationale cover it, an acceptance criterion pins it, and a rejected alternative and a ledger entry record the choice.
- **F4** (grace-elapsed recovery): the done-wait Decision states the resulting state (batten blocked at `Worktree-Teardown`, portal and launchers removed, pair otherwise intact).
  It also names the recovery (`lyx loom commit-records` in the task worktree, then resume batten) and requires the teardown reason to name that recovery instead of `--force`.
  The Open risk now points to it, and an acceptance criterion was added.
- **F5** (archive siblings): the status-commit Decision and its acceptance criterion now cover the friction directory's timestamped archive siblings, with the dirty-weft reason in the rationale.
- **F6** (Add rollback): the archive Decision is retitled and scoped to an existing pair's teardown, `Add`'s rollback is excluded by name, and an acceptance criterion pins the exclusion.
- **F7** (split-run notes): the in-flight Open risk states the loss and why it is accepted, the duplicated auto-mode assumption was removed (Scope Out already records it), and a ledger entry was added.

## Deferred

None.
