# Card 7 — loom-commit-records-verb

**Create:**
- `internal/loomcli/commitrecords.go`
- `internal/loomcli/commitrecords_test.go`

**Edit:**
- `internal/loomcli#Command`
- `internal/loomcli#verbUsesLightweightWiring`
- `cmd/lyx/helptree_test.go`
- `docs/overview.md`

**Uses:**
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#commitStatusDeps`
- `internal/loomcli#loomCommitStatusDeps`

**Intent:**
Add `lyx loom commit-records`, which commits and pushes the run's records from Go when the driver stops.
The driver writes its stop report after its last `lyx shed step`, so no transition commit can capture it, and the Fabric Git Invariant forbids the agent committing it itself: the agent triggers this verb, and the verb commits through `fabricengine`.

`commitrecords.go` holds a `commitRecordsVerb(out io.Writer, d commitStatusDeps) int` over the existing `commitStatusDeps` seam (built by `loomCommitStatusDeps(c.location, c.runID)`, so its commit stages exactly `statusCommitPathspec`: the status file, the review round record, the loom durable directory and the drive-reports directory, each only when present), and the cobra command `commit-records` built by a `(c *loomCLI) commitRecordsCmd()` method:
- `MergeActive` true: success envelope with `committed: false` and a `skipped` reason naming the active merge, exactly as the status seam skips; a probe error is an error envelope.
- `Commit` error: error envelope naming the commit failure.
  A clean tree is not an error: `CommitAnchoredPaths` over an already-clean pathspec is a no-op, so the verb reports success.
- `Push` error: error envelope stating the commit landed locally but was not pushed.
- Otherwise a success envelope.
`Short` is non-empty, `Args` is `cobra.NoArgs`, and `RunE` checks `clihelp.ShouldAbort` first (CLI/Cobra Invariant).
The `Long` text says the ly-drive end-of-session command runs it and that it is safe to run by hand in a task worktree before resuming a batten run whose teardown refused uncommitted run records.

Register the verb in `Command`'s `AddCommand` list and in the loom parent's `Long` text and examples, and add `"commit-records"` to `verbUsesLightweightWiring`: it builds no producer, so a broken module config must not refuse it at the driver's last act.
Add `"commit-records"` to the loom row's `wantSubs` in `cmd/lyx/helptree_test.go`.
In `docs/overview.md`, where the loom verbs and friction notes are described, add the verb and state that friction notes live under `_lyx/loom/friction/` and drive reports under `_lyx/shed/<slug>/drive-reports/`, both committed with the run.

Tests in `commitrecords_test.go`, driving `commitRecordsVerb` over stub `commitStatusDeps` closures with no git: mid-merge skips without calling `Commit` or `Push` and succeeds; a merge-probe error, a commit error and a push error each emit an error envelope with a non-zero exit; the ordinary path calls `Commit` then `Push` once each and succeeds.

**ImpactSummary:** Adds one loom verb and one lightweight-wiring entry; existing verbs and the per-transition seam are unchanged.
