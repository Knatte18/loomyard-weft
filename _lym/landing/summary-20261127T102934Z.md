# Blame a merge fixer only for remote commits it could have added

A merge fixer run at Parent-Align or Finalize ended `pushed` and halted whenever `origin/<task branch>` differed before and after the session.
The halt said the fixer session pushed.
Any other writer of that ref was blamed on the fixer, including a push of the merge commit made before the session started, which halted a run whose landing gate had passed.
The same halts, and the other fixer and fix review halts, named the session's run directory as its log,
but the session runner removes that directory after a session ends `done`, so the halt pointed at evidence that no longer existed.

## What changes

- `internal/landingshed/fixerrun.go`: the fixer loop reads the task head at session start beside the remote tip.
  A moved remote tip ends the run `pushed` only when it is not that head or one of its ancestors, or when the ref was deleted.
  A tip the start head already held is logged and does not change the run's end.
- The `pushed` halt now says the remote ref moved during the fixer session, names the new tip and the start head,
  and its way forward names the range `<start head>..<new tip>`, the only commits the session could have added.
  A deleted ref gets its own wording.
- Fixer and fix review halts name the session as `<engine> session <id>`, which survives cleanup,
  and add `run dir <path>` only while that directory still exists, as after a timeout.
- `Deps.IsAncestor` is a new seam over the task pair, wired in `internal/loomcli/landingdeps.go` through `fabricengine.Fabric.IsAncestor`;
  the seam lists in `loomcli` and `battencli` and the `envkit` fixture name it.
- `internal/gitrepo/push.go`: every push or remote-branch delete that lands logs `gitrepo: push landed` with the checkout, remote, remote ref and new tip.
  The log file's header names the process, so the writer of a moved remote ref can be found.
  Every branch push goes through `recordLandedPush`, so this one line covers them all.

## Design decisions

- Ancestry, not the content-based `HoldsWork`, decides whether a tip predates the session: a commit with the same tree but new history could still be one the session added.
- When the remote tip moved to a commit the session could have added, the halt stays, since only a human can tell whose commits they are; only its wording and evidence change.
- Keeping the run directory for every non-`fixed` end was the other option.
  It would need the session runner to learn the landing row's verdict before its cleanup, so the halt names the session id instead.

## Tests

- `TestFixerRun_Outcomes` gains rows for a remote tip moved to a commit the start head lacks, which halts with the new wording;
  a tip moved to the start head while the turn commits past it, which ends `fixed`;
  and a deleted remote ref.
  Every halt row now asserts the session id, and the timeout row asserts that a run directory still on disk is named.
  The other rows assert that a removed run directory is not.
- The Finalize fix review rows assert the session id in place of the run directory.
- `TestPushBranch_LeavesTraceLine` pushes a branch to a bare remote and asserts the trace line names the ref, remote and tip.

## Merge danger

**Door:** two-way
**Blast radius:** landing
