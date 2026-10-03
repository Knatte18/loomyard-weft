# Pair lifecycle: session-first teardown, origin sweep, retiring driver

Removing a pair now ends its reed session before any worktree goes, finishes a half-removed pair, and keeps late run records in the archive tag.
`--remote` also deletes landed warp branches on origin, and `lyx loom start` no longer adopts a driver strand that is about to remove itself.

## What changes

- New package `internal/pairteardown` owns the sequence: wait for the loom driver to go quiet, run a read-only refusal probe, end the pair's reed session with `reedengine.Engine.Down`, then call `fabricengine.Remove`.
  `lyx fabric remove` and batten's Worktree-Teardown row both call it, so the two paths cannot drift.
  The reason it is a separate package: `reedengine` imports `fabricengine`, so fabric cannot end a session itself.
- `fabricengine.Remove` finishes a half-removed pair instead of answering "worktree not found".
  When nothing of the pair remains it returns a typed not-found error; the CLI reports it as an error and batten treats it as done.
  A non-worktree directory at the pair path is reported and never deleted.
  `RemoveResult` lists the steps the call performed.
- `Remove` commits the sibling worktree's pending run records through the scoped pathspec `fabric sync` uses, before the archive tag.
  The no-force refusal on dirtiness inside that pathspec goes, since it did the sync its remedy asked for.
  Warp-side dirtiness and sibling changes outside the pathspec still refuse without `--force`.
- `remove --remote` deletes the pair's warp branch on origin when its work is landed, using the squash-proof `checkUnlandedWork` rule on the fetched remote tip, and leased on the observed tip.
  A branch that fails the gate is kept with a reason, and `Remove` still succeeds.
- `fabric cleanup --remote` finds leftover warp branches on origin.
  A branch is a candidate when fabric manages it, it is not the default branch or checked out in a hub worktree, it is not an open PR head, and its work is landed.
  Without `--apply` it reports; with `--apply` it deletes candidates leased on their tip.
  Open PR heads come from `internal/githubclient` in `fabriccli`; if GitHub is unreachable, warp-branch deletion is skipped with a reason and the weft sweep proceeds.
- `lyx reed remove --detach` marks the strand retiring before it spawns the remover, and reed's strand status exposes the flag.
  The remover works by guid, and a missing guid is a no-op.
  `lyx loom start` removes a live retiring driver strand and spawns a fresh one.
- The `main-weft` item needs no code change: weft content is per-branch, so it does not advance on landing by design.
  `fabricengine`'s package doc now says so.
- `CONSTRAINTS.md` gains an invariant that every pair teardown goes through the composite, and the Fabric Git Invariant's teardown bullet gains the pending-record commit.
  Docs: `manifest/designs/pair-teardown.md`, `docs/overview.md`, and the fabricengine, reedengine and battenshed package docs.

## Design decisions to check

- The quiet wait is bounded (2 minutes for `fabric remove`) and never kills a live driver.
  On timeout `fabric remove` refuses and names the driver, its `lyx reed attach` command and the `lyx reed down` escape.
  Batten passes a zero wait: it already waited its `driver_exit_grace_s`, so a live driver is ended and recorded as abandoned, as before.
  A parked driver is ended without refusal.
- Refusals run in the probe before the session is ended, so a mistyped `remove` on a dirty pair keeps the operator's strands.
  `Remove` repeats the checks; a refusal only it raises comes from a change made after the probe.
- For a remote tip, "another ref" excludes every copy of the same branch (local and remote-tracking).
  Without that, the branch's own `refs/remotes/origin/<branch>` would always make it look landed.
- The retiring mark records only that someone requested removal; it does not prove the records were committed.
  A failed detached-remover spawn clears the mark.
- Batten's half-removed special-casing (`PairSiblingRemnant`, the `RemovePairBranch` re-entry) and the fabric helpers it left unused are deleted.
- Known limits: concurrent teardowns of one pair are not serialized (batten holds the prime lock, `fabric remove` does not).
  A squash-landed branch whose lines default later edited reads as unlanded and is kept.
  Reed's watchdog is not taught to reap a session whose worktree path is no longer a registered worktree.

## To check by hand

The run's integration stage passed with no failures.
Not run by hand against a real remote: `lyx fabric remove --remote` and `lyx fabric cleanup --remote --apply` on a hub with real leftover warp branches and GitHub reachable, then once with GitHub unreachable.
