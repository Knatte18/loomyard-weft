# Gate: run the integration tier once and keep test cache keys stable

A verify on OSL-1046 linked two sets of test binaries that Cortex XDR holds for 8-10 s each, and a re-run on an unchanged tree ran many packages again.
This change makes verify run the `integration` tier alone, keeps per-invocation values out of what go test reads, and runs a failed test before the full command.

## What changes

- `CLAUDE.md`'s push rule names one run, `lyx gate test --tags integration ./...`.
  The `integration` tag compiles every untagged test too; the only `!integration` files are TestMain shims whose tagged TestMain does the same setup and more, so the unit run was pure repetition.
- The loom plan's `## verify:` follows the same rule.
  `pattern/PATTERN-test-speed.md`'s row for Webster's gate, Publish and Finalize now reads build, vet under each tag, then the `integration` tier alone, and the example `verify_tiers` in `internal/languageconfig`'s doc drops the untagged `go test` the same way.
  Plan writers followed the old row, so every loom run's Webster gate, Publish and Finalize ran the unit tier twice.
- `lyx gate test` removes `LYX_TRACE_ID` from go test's environment, beside the strand name it already removes.
  Every `lyx` process mints a new trace id, so each gate run changed the cache key of every package whose code reads it through `internal/logger`.
  `logger.TraceIDEnv` now names the variable, and `reedengine` and `shedverbs` use it instead of the literal.
- Inside a hub, a tagged `lyx gate test` publishes the prebuilt `lyx` into a store under the gate-slot directory, named by the binary's SHA-256, and exports that path.
  Before, the path was a fresh directory per run, so every package reading it through `internal/testkit/lyxbin` missed the cache on every tagged run.
  The path is stable while the tree is unchanged and changes with the binary, so a cached result is never reused for a changed `lyx`.
  Outside a hub the per-invocation path and its cleanup stay.
- The loom verify gate (`internal/loomshed`), in both its round and whole-diff forms, runs a Publish failure record's failing tests before the told or derived command.
  The `tmux` pass over the impacted packages stays after it.
  A failure that still holds is known in seconds, and the full tier runs only once it passes.
- A test in `internal/shedbuild` no longer writes and removes a fixture directory, which changed the `internal/` listing that the source scans read.
- `docs/benchmarks/gate-test-xdr.md` records the before and after on OSL-1046: a re-run on an unchanged tree ran 0 of 162 packages and took 26.6 s, against 186.0 s for the former two-tier verify.

## Design decisions

- Each prebuild prunes store entries other than its own whose time is older than two hours, and every reuse refreshes the entry's time.
  A gate run whose binary is pruned while it still runs fails its remaining `lyxbin` tests with the stale-prebuilt error; the retention is far longer than a run.
- The store path derives from the slot pool (`PrebuiltDir`), never a new `.lyx` literal, and the store is ephemeral hub state, never tracked.
  The prebuild stays inside the held slot, and the build output still goes to the process run directory of `internal/lyxtemp`.
- Publishing stages a `.partial-` file and renames it into place; a lost rename race keeps the winner's entry.
- Stripping the trace id means a test process mints its own instead of joining the gate run's.
  Logging under go test is off unless `LYX_TRACE` is `1`, so only an operator-requested trace loses the correlation.
- Out of scope: scoping verify to touched packages (`verify-diff-scope`), and cache hits across worktrees for tests that open files of the tree, which Go keys on the file's absolute path.

## To do by hand

- The hub's `language.yaml` sets no `verify_tiers`; a hub that sets them should list the `integration` tier alone, with no untagged `go test` before it.
- Check `internal/gatecli/prebuild_test.go` `TestStorePrebuilt` for the store's race and error paths: a pre-published entry under the content name returns the same path and leaves no `.partial-` file, and an uncreatable store makes `storePrebuilt` fail.
  The review rated their absence MEDIUM and the tests were added late.
- Check `storePrebuilt` in `internal/gatecli/prebuild.go`: when `os.Chtimes` on an existing entry fails with not-exist because a concurrent prune removed it, it must publish the entry again instead of failing the run.
- Check that a tagged run outside a hub exports the build's own `-o` path and removes its directory afterwards (`internal/gatecli/test_integration_test.go`).
- Check that `internal/shedverbs/loop.go` `childEnvironment` and its test use `logger.TraceIDEnv` at every site.
