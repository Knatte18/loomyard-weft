# landing: stop blaming the fixer for remote ref moves it did not make

The landing fixer loop ended a run as `pushed` whenever `origin/<task branch>` differed before and after the session,
and the halt said the fixer session had pushed.
Any other writer of that ref was blamed on the fixer,
including a late push of the merge commit made before the session started.
The halt also pointed at a session run directory that had already been removed.

- The loop now reads the task head before the session starts.
  A remote tip that moved to that head or to one of its ancestors is logged and no longer halts the run.
- A move to a tip the starting head does not hold still halts.
  The halt says the ref moved during the session, not that the session pushed,
  and asks to check the commits `<start head>..<new tip>`.
- A deleted remote ref still halts, with its own wording and way forward.
- Fixer and fix review halts that printed `session log: <run dir>` now print `session:` with the engine and session id.
  The run directory is named too only while it still exists, as after a timeout.
- Every branch push, move or delete that lands through `internal/gitrepo` now leaves a `gitrepo: push landed`
  trace line naming the checkout, remote, ref and new tip (empty for a delete),
  so the writer of a remote ref can be found from the logs.

Closes #271
Closes #272

## Design decisions

- Attribution rests on ancestry, not on who pushed, which is not known.
  One consequence: if the session itself pushed a commit that the task head already held at its start,
  that push is no longer reported.
- The ancestry query is a new `Deps.IsAncestor` seam in `internal/landingshed`,
  wired in `internal/loomcli` to the fabric pair's `IsAncestor`.
  A failing query is returned as an error rather than read either way.
- The run directory is not kept after a done session; the halt names the session id, which outlives it.
- The trace line does not identify the process behind the push reported in #271.
  It covers later pushes made through `internal/gitrepo`, not a plain `git push` run outside it.

## To check by hand

- The argument order of the ancestry query matters:
  swapping the call to `IsAncestor(startHead, remoteAfter)` would hide a push of the session's own commits.
  The case "a remote tip that moved back to an ancestor of the start head is no push" in `TestFixerRun_Outcomes`
  (`internal/landingshed/fixerrun_test.go`) is meant to fail under that swap, which has not been tried by hand.

## Merge danger

**Door:** two-way
**Blast radius:** landing

Outside landing, the push paths of `internal/gitrepo`, which every caller shares, gain a log line and nothing else.
