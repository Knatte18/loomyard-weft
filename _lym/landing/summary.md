# Close the webster reset records-push report with no code change

This change has an empty diff.
The behavior issue #293 asks for is already on `main`, from 3fb0787e3 (Retry pushes quietly and land without waiting on a push),
when the reset's records commit returns no error.
One failure path stays open; see "To check by hand".

The issue reported that `webster reset` only committed its records and left their push to the background pusher,
which writes no owed record, so a failed push did not hold the run.
On `main`:

- When the records commit returns no error, `webster reset` pushes the pair's records synchronously
  through `pushPairRecords` and `pairteardown.PushRun`.
  On the error path under "To check by hand", it does not.
- A push it cannot land leaves an owed record that holds the run, and the verb reports it in `push_error`.
- `pattern/PATTERN-pair-remote.md` and `pattern/PATTERN-run-fsm.md` describe this behavior.

Closes #293

Merging closes the issue with that failure path still open, and this change files nothing to track it.
To keep it tracked, file a new issue before merging.

## Design decisions

The issue also suggested checking at plan review that every decision item is covered by a card.
That is not part of this change.

## To check by hand

- A partial records commit skips the reset's synchronous push, at `internal/webstercli/reset.go:193`.
  When the records commit lands but `Fabric.CommitWithTrailers` still returns an error,
  the verb refuses before it reaches `pushRecords`.
  The commit carrying `Run-Transition` is then left to the background pusher,
  so an unreachable origin writes no owed record and does not hold the run.
  This is the reported gap, narrowed to one failure path, and it stays open.

## Merge danger

**Door:** two-way
**Blast radius:** none
