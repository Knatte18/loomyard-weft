# Fast batten start, local-only fabric pairs and one Command producer

All ten batches reported `status: done`.
No batch needed recovery, and no digest carried a `Pattern-Departure:` trailer (`departures` was empty throughout).
The digests carried no warnings.
The scope guard's `deviations` lists are informational; each is named below.

- **01 (cards 1–3), head `6d8423fc7`:** the before measurement of a batten start, the per-branch local mark in `internal/gitrepo` with the push primitives refusing a marked branch, and fabric's pushes reporting a local branch as skipped.
  Deviations: `cmd/lyx/gitrepoboundary_test.go`, `internal/fabricengine/pushanchored.go`, `internal/gitrepo/branchmark.go`, `internal/gitrepo/push.go`.
- **04 (card 4), head `0ea24c5e3`:** `lyx fabric add --local`, and a plain add adopting a marked warp branch and clearing the mark.
  Deviations: `internal/fabricengine/add.go`, `internal/fabricengine/add_local_integration_test.go`.
- **05 (cards 5–6), head `f8a24545b`:** a local pair's remove runs no origin step and drops its records, and cleanup of a local pair's orphan weft branch pushes nothing.
  Deviations: `internal/fabricengine/destroy.go`, `internal/fabricengine/remove_local_integration_test.go`.
- **07 (card 7), head `ac8e186bc`:** one spawn entry for the status push with the prime rule, slug roles, `_board` cwd and the local-mark check.
  Deviations: `cmd/lyx/main_test.go`, `internal/fabriccli/localpair_integration_test.go`, `internal/fabricengine/commit_integration_test.go`, `internal/fabricengine/export_test.go`, `internal/fabricengine/spawn.go`.
- **08 (card 8), head `6b1e8b696`:** the push lock is try-locked by detached children, with per-side pending flags.
  Deviations: `cmd/lyx/destructiveguard_test.go`, `internal/fabricengine/coalesce.go`, `internal/fabricengine/coalesce_test.go`, `internal/fabricengine/pendingflags.go`, `internal/fabricengine/pushanchored.go`.
- **09 (card 9), head `716cf2147`:** a per-slug teardown lock between Remove and the pair's pushes.
  Deviations: `internal/fabricengine/teardownlock.go`, `internal/fabricengine/teardownlock_integration_test.go`.
- **10 (card 10), head `0d505e0fa`:** batten's, loom's and darn's status seams and batten cleanup spawn the detached push.
  Deviations: `internal/battencli/cleanup.go`, `internal/battencli/cleanup_integration_test.go`, `internal/battencli/cleanup_test.go`, `pattern/PATTERN-push-both-sides.md`.
- **11 (cards 11–14), head `163c48eb5`:** literal-argument quoting in `verifyrun`, `lyx ide seed` and `lyx ide open`, `lyx reed window`, and the `Command` producer.
  Deviations: `internal/idecli/cli.go`, `internal/reedcli/window.go`, `internal/reedcli/window_test.go`, `internal/shedadapters/command.go`.
- **15 (card 15), head `249bd4dfb`:** the `Command` engine replaces the switched rows and engines.
  Deviations: `internal/battencli/cli.go`, `internal/battencli/lifecycle_integration_test.go`, `internal/configreg/configreg_test.go`, `internal/shedrecipe/entries_command.go`, `internal/shedrecipe/entries_command_test.go`.
- **16 (card 16), head `6c81b1200`:** the after numbers; the measurement showed no overhead to cut, so no production code changed.
  Deviations: `internal/battencli/lifecycle_integration_test.go`.

The plan-level verify gate runs after this turn ends.
