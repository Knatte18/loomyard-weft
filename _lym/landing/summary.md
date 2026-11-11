# adopt: pass pauses, wait on push locks, clear lym files, rejudge stops

`lym adopt` had three faults that together left an operator unable to take a hub over after a pause or a partial adopt.
This change fixes them in `adoptcli` and `fabricengine`.

## What changes

- **A pause no longer holds an adopt.**
  A hub-wide `lym tender pause` left from another machine made the adopt worker's first network call wait silently.
  `runAdopt` and `runWorker` now set `pauserecord.SetGateExemptAll` on their own process's network gate and restore it on return.
  Processes the worker starts, such as a dobby or an adopt check, install their own gate, which a pause still holds.
- **The reconcile waits on a short-lived push lock.**
  After `lym tender resume hub`, the deferred records push it spawns holds the weft push lock, and the reconcile stopped on `main-weft (busy)`.
  `ReconcileOptions` gains `PushLockWait`, and a records branch move retries its locks once a second while only the push lock is held.
  The adopt sets the bound to 30 seconds.
  A zero bound keeps the old behaviour: a held push lock reads busy at once.
  A held write lock or prime lock still reads busy at once.
- **`--drop-local` no longer fails on untracked lym files.**
  `git reset --keep` refused to move a weft whose worktree held an untracked file, such as `_lym/reviews/code/generation-record.json`,
  that origin's tip tracks.
  After the tip is archived, `DropLocal` removes each such untracked, unignored file of a records worktree and then resets.
  Each removal is recorded as a `path_removed` entry.
  A code branch's untracked files are never removed.
- **A stopped record at `done` is judged afresh.**
  A new `lym adopt <subject>` over a record that stopped at `done` replayed that record's stale `gone` and `needs_replay` stops.
  `rejudges` now restarts such a record at `reconcile` with its steps cleared.
  The `--keep-local` and `--drop-local` re-runs behave the same way, so the stop is judged afresh.

`docs/overview.md` and the `adoptcli` and `fabricengine` package docs describe the new behaviour.

## Design decisions

- The pause exemption covers every pause entry for the adopt's own process rather than refusing the call at once.
  It is scoped to that process's gate, so the worker's children are unaffected.
- The push-lock wait releases both locks between tries, so the push holding the lock can take the write lock in the meantime.
- Untracked-file removal goes through the path-ownership gate with a new `ownedUntrackedTrackedAt` predicate.
  It passes only a file the worktree's index does not track and that the origin tip does, so nothing else can be removed.
  The archive tag keeps the old local state.
- Restarting a `done`-stopped record at `reconcile` costs a repeat of the steps that had finished.
  That is the price of judging the hub afresh rather than trusting the recorded branch states.

## Check by hand

No regression check ran for this task, and the following was not run against a real hub:

- `lym tender pause` the hub, then `lym adopt root`: it should proceed with no wait on the pause.
- `lym adopt root --drop-local` on a weft with an untracked file that origin's tip tracks: it should archive the tip, remove the file and move the branch.
- After a partial `--drop-local`, remove a pair with `lym fabric remove`, then run `lym adopt root`: it should not report the removed pair's old stops.
