# Gate: verify runs the integration tier once, re-runs hit the cache

A verify on a Cortex XDR machine linked and ran more test binaries than it needed.
The command ran the unit tier and then the `integration` tier, which compiles and runs every untagged test too, so each unit test ran twice.
A re-run on an unchanged tree also repeated packages that had passed, because per-invocation values in go test's environment changed Go's test cache key on every run.
After a Publish failure, the verify ran the failing test only after the full tiers.

## What changes

- Verify runs the `integration` tier alone.
  `CLAUDE.md`'s push rule, the `PATTERN-test-speed.md` Webster gate/Publish/Finalize row, the `internal/languageconfig` example `verify_tiers` and the command `internal/impactset` derives for an impacted set no longer run a separate untagged tier.
- `lyx gate test` removes `LYX_TRACE_ID` from the environment it gives go test, beside the strand name it already removes.
  Every `lyx` process mints and exports a new trace id, so every package that logs missed the test cache on each run.
  The variable name is now the exported constant `logger.TraceIDEnv`, which the other sites that read or strip it use.
- Inside a hub, a tagged `lyx gate test` publishes the prebuilt `lyx` into a store under the gate-slot directory (`Pool.PrebuiltDir`), named by the binary's SHA-256, and exports that path.
  The path was a fresh directory per run, so every package that runs `lyx` through `internal/testkit/lyxbin` missed the cache.
  Outside a hub the per-invocation path stays.
- The loom verify gate in `internal/loomshed`, in its round and whole-diff forms, runs a Publish failure record's failing tests before the told or derived command, and the `tmux` package pass after it.
  A failure that still holds ends the run in seconds.
- `internal/shedbuild`'s load test named a directory of the tree, where it wrote and removed a fixture, so every source scan that lists `internal/` saw a changed directory.
  It now names one of the package's own files.
- `docs/benchmarks/gate-test-xdr.md` gains the before and after runs on OSL-1046.
  An `integration` re-run on an unchanged tree ran 0 of 162 packages and took 26.6 s, against 186.0 s for the former two-tier verify.

## Design decisions

- No test is excluded by the `integration` tag: the only `//go:build !integration` files are TestMain shims whose tagged TestMain does the same setup and more.
  The tagged run therefore reports a unit failure too.
- The prebuilt store is content-addressed, so the path is stable while the tree is unchanged (the build runs with `-trimpath`) and changes whenever the binary does; a cached result is never reused for a changed `lyx`.
  A binary already in the store is reused and its time refreshed, and each prebuild prunes other entries older than a fixed retention.
  The store is a hub-level ephemeral directory derived from the pool's directory, within the held gate slot.
- Dropping the trace id has one cost: a test process mints its own id instead of joining the gate run's.
  Logging under go test is off unless `LYX_TRACE` is `1`, so only an operator-requested trace loses the correlation.
- A gate run whose binary is pruned while it still runs fails its remaining `lyxbin` tests with the stale-prebuilt error.
  The retention is far longer than a gate run, and every use refreshes the time.
- Out of scope: scoping verify to touched packages (note `verify-diff-scope`), and cache hits across worktrees for tests that open files of the tree, which Go keys on absolute path.

## To check or run by hand

- The hub's verify command lives in the board's `darn.yaml`, outside this branch, and is changed through `lyx config darn`; confirm it reads `lyx gate test --tags integration ./...`.
- Some packages may still miss the cache through other per-invocation inputs, such as `LYX_GATE_TEST_PID` and the slot variable that `internal/gatecli`'s own tests read.
  The benchmark records the packages run per run; re-run a second `lyx gate test --tags integration ./...` on a clean tree to see what remains.
- The measured numbers come from OSL-1046 only; a refill run after this change rebuilds every package importing `internal/logger`, so expect a slow first verify.
