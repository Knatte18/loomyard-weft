# Round 4 fixer report — plan

## Changes

- **F1 (MEDIUM), fixed differently in part.**
  `_lyx/plan/08-push-lock-pending-flags.md`: the `pending.code` respawn rows of `TestCoalesceFlagged_PendingFlags` become one table-driven group, in the same test, generated over both flag sides, with rows (a) to (f) per side as the fix proposed.
  Rows (b) to (f) each change one field of (a)'s valid flag, or its bytes.
  The bounds paragraph gains the disposition of a flag that does not decode.
  The proposed change I did not make: the fix's records side was "a holder owing code under a code path that is not the paired worktree".
  No real caller is such a holder: every caller that owes `code` that way also owes `records`, so `pending.records` would only be taken, never respawned.
  The side now uses the reachable holder the review's Issue names: a code-only `CoalescePushRequest` child under the paired code worktree.
  The proposed "one of an owed side is only removed" also became "the holder ignores `takePending`'s decode error and treats the flag as set".
  Silently dropping an owed-side flag could lose the re-step it signals.
  Two further changes close the half-written-flag race, because dropping such a flag could otherwise lose a push:
  - `markPending` writes through `fsx.AtomicWriteBytes`, which is added to Uses.
  - `takePending` renames a flag to a taker-private name before reading it, so the atomic rename never meets a Windows sharing violation, and it reports a decode error as distinct from a rename, read or remove failure.
  A row was added for an owed-side flag that does not decode, written by the step after `markPending` has run.
- **F2 (LOW), fixed as proposed.**
  In `_lyx/plan/15-command-engine-replaces-switched-rows.md`, the `TestCommandEntry` bullet now names one local env helper in `internal/shedrecipe`'s tests.
  The helper starts from `newTestEnv` and sets `Setting` (to `fakeSetting`), `Command.Run` (to a recorder), `Command.LogPath` and `Command.Dir`.
  The call rows use no `{task_dir}`.
  `minimalEntries`' `Command` row uses the same helper, in the Tests bullet and in the Recipe paragraph.

`lyx loom validate-plan` returns `ok: true` after the changes.

## Self-check

- fix-affected-set: no findings.
- fix-consistency, MEDIUM: the owed-side corrupt-flag row was unreachable, because `markPending` rewrites the flag first. Fixed as proposed: the step writes the corrupt bytes, and the row expects one more step.
- fix-consistency, LOW: the `takePending` sentence (decodes every flag) and the bounds sentence ("never decoded") disagreed. Fixed: the bounds sentence now says the holder ignores the decode error and treats the flag as set.
- fix-claims, MEDIUM: the same unreachable row as the fix-consistency MEDIUM, merged with it and fixed there.
- fix-claims, MEDIUM: the same contradiction as the fix-consistency LOW, plus the loose "never loop" claim, merged with it. Fixed by the same rewording, which drops the loop claim.
- fix-claims, LOW: on Windows, `markPending`'s atomic rename can fail while a reader holds the flag open. Fixed with my own change: `takePending` renames the flag to a taker-private name before opening it. Retrying the rename was not chosen, because the repo has no rename-retry helper and a retry only shrinks the window.
- fix-proposal, LOW: the records side of the matrix used an unreachable holder. Fixed as proposed: a code-only child under the paired code worktree.
- fix-proposal: no case found against F2's proposal.

## Disputed

None.

## Deferred

None.
