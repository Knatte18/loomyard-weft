# Unblock Webster's run-exit audit on Master's friction note

A Webster run with every batch done could end stuck with no way forward short of `lym webster reset --to start`, which throws away every batch.
Three faults combined to cause it:

1. **The parent-write audit flagged Master's friction note.**
   `CheckParent` allowed Master to write only `outcome.yaml` and `summary.md`.
   The Master prompt tells Master to write a friction note under `_lym/loom/friction/`, and that write counted as a correctness finding.
   Run now saves the note path it puts in Master's prompt as `State.MasterNotePath` at spawn.
   `CheckParent` takes that path and skips a write to it, whether the transcript spells it as told or through the `_lym` link.
   Both audit sites read the path from state: record-batch's whole-session parent audit and the run-exit audit.
   The path is stored, not recomputed, because `friction.NotePath` picks a fresh `-2` name once the note exists.
   The Master stencil's "never write anything but the contract files" rule now names the friction note too.

2. **A verify-gate fixer's commit made `accept-audit` refuse.**
   `AcceptPendingAudit` accepted only clean parent merges above the last batch head, and a fixer's commit is not one.
   The verify gate now records the HEAD it passes at as `State.FixedHead`, but only when fixes moved HEAD above the pre-fix head.
   It records nothing on a pass with no fixes.
   `accept-audit` first checks HEAD against the last batch head.
   If that refuses, it checks HEAD against `FixedHead` instead, but only when the last batch head is an ancestor of `FixedHead`.
   The commits between the two are the fix commits that the gate's own commit check already vetted.
   A commit above `FixedHead` that is not a clean parent merge still refuses, naming `reset --to last-batch-head`.
   `lym webster reset` clears `FixedHead` the same way it clears `PreFixHead`.

3. **`accept-audit` could not check a path on the records side.**
   In a hub, `_lym` links into the separate records worktree.
   So a pending path such as the friction note resolved outside the task worktree and was always uncheckable.
   A path under the run's `_lym`, outside webster's run, reports and plan directories, now clears when it is absent.
   It also clears when git, run in that directory, neither ignores it nor sees it differ from HEAD (`recordsSideStatus`).
   A records-side path that has changed refuses, and its way forward is to restore it to the records side's last commit, then `accept-audit`.
   The way forward in the run-exit stuck reason and the run-entry refusal (`pendingPathsWayForward`) now gives this records-side route,
   so neither sends such a path to the reset-to-start route.

## Design decisions

- `FixedHead` stays as it is after a pass with no fixes, so a later clean run does not drop the earlier fixes' tolerance.
  If HEAD is later reset below it, the ancestry check keeps it from tolerating anything.
- Records-side acceptance excludes webster's own run state: state.json, the reports and the plan.
  Those keep their existing checks or the reset route, since a forged state file could pass as "unchanged" once a sync commits it.
- The records-side check runs read-only git (`diff --quiet`, `status --ignored`, `ls-files`) through the existing `Git` seam.
  The fabric-boundary entry exempts read-only verbs.

## Tests

- `TestCheckParent` gains rows for these cases:
  - a write to Master's friction note is allowed, by absolute and by relative spelling;
  - a write to another friction note is still a parent-write.
- `TestAcceptPendingAudit_AcceptsVerifyFixCommits` accepts a fix commit up to `FixedHead`, and refuses once a commit sits above it.
- `TestVerifyGate_*` (the pre-fix head test) asserts that a pass after HEAD moved records the passing HEAD as the fixed head.
- `TestAcceptPendingAudit_RecordsSidePath` links `_lym` to a second repository, then checks each case:
  - an absent note and a committed, unchanged note both clear;
  - an edited note and an uncommitted new note both refuse toward a restore;
  - a path under webster's run state still takes the reset route.
  It checks both the `accept-audit` refusal and the pending way forward.

## Merge danger

**Door:** two-way
**Blast radius:** webster
