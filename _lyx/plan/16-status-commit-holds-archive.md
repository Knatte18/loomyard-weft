# Card 16 — status-commit-holds-archive

**Edit:**
- `internal/loomcli/wiring.go`
- `internal/loomcli/wiring_commitstatus_test.go`
- `internal/loomcli/wiring_commitstatus_integration_test.go`

**Intent:**
Keep loom's status commit from landing half of a rework round, so the archive's moved-from deletions, its copies, the classed `record.json` and the new plan land in PR-Rework's one round commit on every path, interrupted ones included (D14).

Shed calls the status seam on every persist: a Stuck PR-Rework (its gate out of attempts), a Failed one (a returned error, card 5's archive collision and a `Commit` seam error included), a Paused one, and the `running` write a resume makes before re-calling it.
`statusCommitPathspec` adds the reviews root and the loom durable directory whenever each holds a file, and `git add -- <path>` stages deletions too, so a status commit anywhere between card 5's archive step and its round commit would land `round-<N>/` (its `record.json` and every `prior-generation/` copy) and the review run directories' deletions, while the `_lyx/plan/` deletions, the new plan and the webster deletions wait for the round commit.
The window runs to the round commit itself, not to the class write: card 5 step 4 writes `class` into `record.json` before it commits, and a status commit that lands a classed record makes `committedRejections` count the round, so the next step takes `finish(true)` and clears the rejection with no round commit at all.

- `statusCommitPathspec` holds the round back for as long as a rejection is pending: while `loomengine.LoomRejectionPath` exists, or a stat of it fails for any reason other than not-exist, it returns the status file and the drive reports only, leaving the reviews root and the loom durable directory out.
  The pending rejection is the right marker because `lyx loom reject` writes it before the archive step starts and PR-Rework removes it only after its round commit has landed, so it covers the archive, the session, the class write and the commit with no round reading.
  The round commit's pathspec (card 5) covers both held directories, and once PR-Rework clears the rejection, the next status commit sweeps the loom durable directory's remaining files (friction notes, say) again, so nothing is lost, only deferred.
  The doc comments of `statusCommitPathspec` and `loomCommitStatusDeps` say so.

Bound: the hold needs a pending rejection, which only `lyx loom reject` writes and only PR-Rework clears after its round commit; it defers the sweep of `_lyx/loom/` and `_lyx/reviews/` and cannot drop a file from it.
A stat error holds back rather than sweeps, since holding back defers and sweeping could split the round.

Tests: `wiring_commitstatus_test.go` gains table cases: a pending rejection leaves out both directories though each holds a file, both with an unclassed highest round and with a classed but uncommitted one (the state a `Commit` failure after the class write leaves); no pending rejection keeps today's pathspec.
`wiring_commitstatus_integration_test.go` (real git) moves a review run directory into a round whose `record.json` carries a class, with a rejection pending, runs the status seam and asserts HEAD still holds the review run directory and no `round-<N>/` file, then removes the rejection and asserts the next status commit records both.

**ImpactSummary:** one narrowing branch in `statusCommitPathspec`, local to loomcli; outside a pending rejection every status commit stages exactly what it does today.
