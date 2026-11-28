---
round: 1
focus:
  - "`hubWorkOriginLacks` and `checkResetTarget` in `internal/fabricengine/clone.go`: which work a `--reset` can still destroy that origin does not hold (local-only tags, detached HEADs, stashes, uncommitted edits, the records store, the Bolt checkout, worktree layouts the scan does not match), and whether each hub a killed clone can leave still passes the refusal."
  - "`--reset` advice, across `existingHubError` and every other place that names a `--reset` clone: does each name it only where unpushed work is accounted for, does the `.git`-at-Bolt test separate a whole hub from a partly cloned one, and is the printed `mv` way forward runnable and safe for the paths and linked worktrees involved?"
  - "`deleteLeftoverUnderLock` in `internal/fabricengine/cleanupremotewarp.go`: do the tags `pushLocalArchiveTags` pushes cover every way the landed check can count a local-only tag as holding the candidate's tip, and what happens on a failed or partial push, a dry run, and under the held teardown lock?"
  - "`remover.run` and `settleGonePair` in `internal/fabriccli/remove.go`: for each verdict, with and without `--remote`, does a gone pair with a pending record get settled as the record's re-run requires (including a failed tag push, a held lock, records naming another command, and the reported result fields), and does `ErrPairNotFound` still reach the caller where it should?"
  - "Authentication refusal on a leased push (`internal/gitrepo/push.go`, `seedPushOutcomeUnknown` and `seedPushCause` in `internal/fabricengine/destroy.go`): can a matched trigger line coexist with an outcome origin may have applied, how does a refusal on a later seed ref treat what an earlier push already landed, and do other callers of `UpdateRemoteBranchLeased` behave correctly with the added sentinel?"
  - "The removed pending-record clearing in `RunTeardownChild` and the removed way-forward condition in `recordsArchiveWayForward`: can any path now leave a record that nothing clears, or name a re-run that cannot finish?"
  - "The `Loosened-Check:` and `Root-Cause:` trailers against the diff and the changed tests: is the cause measured, does the renamed test row still check what its name says, and does each of the six faults in the board entry have a test that would fail on the base code (the catch-up call site in `internal/battencli/catchup.go` included)?"
  - "`_lym/landing/summary.md` against the diff: are its claims borne out, and does its `## Merge danger` door and blast radius cover everything the diff can break?"
---

The decision record holds only `## Goal`: six faults to confirm with a test that fails first, and to fix where confirmed.
No review input or amendment applies.
The entries follow the faults, the two that gate a deletion first.
