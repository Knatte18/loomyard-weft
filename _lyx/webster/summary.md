# Batten takes over landing mechanics and retries

Both batches reported `done`; the run is complete.

## Batch 01-gitrepo-push-retry (head `d063c46eb`)

Cards 1 to 6 landed.
gitrepo's plain push primitives retry a remote `(failed)` or `cannot lock ref` refusal and wrap an exhausted one in a sentinel.
Fabric's branch pushes and the archive tag push use that retry, and the coalesced push warns on an exhausted one.
The Publish failure record carries its target producer and write time, and goto's held-run-lock refusal carries a kind batten can read.
`landing.yaml` gained the hub-wide `deploy` key, and Run-Shed no longer notices the orch of a done child.
The `internal/gitrepo` test budget row was raised from 41 to 44.
The digest listed no deviations; its warnings were informational scope notes on helper symbols in `internal/gitrepo` and `internal/fabricengine`.

## Batch 07-row-failure-notice (head `d34668b2a`)

Cards 7 to 11 landed.
Worktree-Teardown notices the orch once per failure episode through a row notifier the Land row reuses.
The Land producer covers the land lock, prime check, pull, deploy, prune and ready notice, and the Land row is registered, wired, guarded and placed between Run-Shed and Worktree-Teardown.
Run-Shed sends a child blocked on its first Publish verify failure to the record's target and resumes it.
The orch role stencil, `CLAUDE.md` and the refusal spec move each landing step to its new owner.

Reported deviations from the declared file-ops:
- Fixes to batch 01 leftovers: `cmd/lyx/gitrepoboundary_test.go` (pinned method list) and gofmt in `internal/landingshed/verifygate.go`, `internal/loomcli/darnwiring.go` and `internal/loomcli/landingdeps.go`.
- New wiring files: `internal/battencli/land.go`, `internal/battencli/publishretry.go` and `internal/battencli/publishretry_test.go`.
- Guard and fixture updates: `cmd/lyx/verifiedtree_test.go`, `pattern/PATTERN-verified-tree.md` (allowing `internal/battencli/land.go` to call `verifyrun.Run`), `internal/battencli/run_test.go`, `internal/battencli/wire_test.go` and `internal/shedrecipe/entries_batten_test.go`.

The fork also reported using `sed -i` once, for an import insertion in `internal/battenshed/teardown_test.go`, against the editing rule.

## Verify gate fixes

The gate's first attempt failed on `TestWire_EverySeamFilled` in `internal/loomcli`: seventeen new `InnerRun`, `Teardown.Notices` and `Land` seams were nil after `wire()`.
The fixer fork reports that `battencli` fills these seams, so `wire` leaves them nil on purpose.
It listed all seventeen in `intentionallyNil` in `internal/loomcli/wiring_seams_test.go`, each with the same "batten-only seam, filled by battencli" reason as the existing batten-only entries, and weakened no assertion.
Its commit is `6cd71c73f`, and it reports `lyx gate test --tags integration ./internal/loomcli` passing with a clean worktree.
