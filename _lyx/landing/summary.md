# Keep gate test results cached across re-runs and run a failed test first

A verify on an unchanged tree now takes every package from Go's test cache.
A verify after a Publish failure runs the failing test before the full command.
On OSL-1046 the re-run of the integration-only verify took 26.6 s with all 162 packages cached.
The two-tier verify before took 186.0 s with 99 of 324 package runs cached.

## What changed

- **Verify runs the `integration` tier alone.**
  The tag compiles every untagged test too.
  The only `//go:build !integration` files are TestMain shims whose tagged TestMain does the same setup, so the separate unit run only cost its own wall time and test binaries.
  The hub's `darn.yaml` verify, which is hub-wide config on Bolt, was changed by the parent with the operator's approval.
  This branch changes `CLAUDE.md`'s push rule to match.
- **`lyx gate test` drops `LYX_TRACE_ID` from go test's environment** (`internal/gatecli/run.go`).
  Every `lyx` process exports a new trace id, and Go keys a cached test result on every variable a test reads, so each package that logs missed the cache on every run.
  `logger.TraceIDEnv` now names the variable.
- **Inside a hub the prebuilt `lyx` of a tagged run lives in a content-addressed store** (`internal/gatecli/prebuild.go`, `gateslot.Pool.PrebuiltDir`).
  The build still goes to the process run directory.
  It is then copied into `<gate slot dir>/prebuilt/lyx-<16 hex digits of its SHA-256>` through a per-process staging file and a rename, and `gateslot.PrebuiltLyxEnv` names that copy.
  An unchanged tree therefore exports the same path every run, since the `-trimpath` build is reproducible, and a changed `lyx` gets a new path, so no cached pass outlives it.
  A stored binary is reused and its modification time refreshed, and each store prunes the other entries unused for two hours.
  Outside a hub the per-invocation path stays.
- **A shedbuild test no longer writes into its package directory** (`internal/shedbuild/load_test.go`).
  It created and removed a fixture there, which changed `internal/shedbuild`'s modification time on every run, and every source scan that lists `internal/` missed the cache.
  It now uses the package's own `doc.go` as the relative path that resolves.
- **The loom verify gate runs a Publish failure's failing tests first** (`internal/loomshed/verifygate.go`, `publishfailure.go`).
  `publishFailureSteps` now returns the failing-test steps and the `tmux` package pass separately.
  The gate puts the failing tests before the told or derived command and the package pass after it, in both the round and the whole-diff form.
  `PATTERN-test-speed.md`'s gate table says the same.
- **`docs/benchmarks/gate-test-xdr.md`** gains a section with the before and after runs and the three causes of the misses.

## Design decisions

- The store sits in the hub's gate-slot directory, outside every worktree.
  Go checks only files inside the module root, so the store's own churn never changes a test's inputs.
- The store is pruned by last use, not by count.
  A gate run whose binary is pruned while it still runs would fail its later lyxbin tests with the stale-prebuilt error, and two hours is far past any gate run.
- The trace id is removed rather than pinned to a constant, so no test log carries a trace id that looks real.
  A test process mints its own; logging under go test is off unless `LYX_TRACE` is `1`.

## Testing

- `internal/gatecli`: the tagged-run subtest now checks the exported path is a `lyx-` entry of the store holding the build's content.
  It also checks that a second run of the same build exports the same path, that a changed build exports another, and that the build directory is removed.
  The first subtest checks that the trace id is dropped.
  The stand-in go now writes its `-o` file.
- `internal/gatecli/prebuild_test.go` (new, untagged) covers the content name, reuse with a refreshed time, and pruning past the retention, under an injected clock.
- `internal/loomshed`: the step table and both integration expectations now pin the failing tests first and the package pass last.
- Module-wide: `lyx gate test --tags integration ./...` passed four times with the branch's `lyx`, the last with every package cached.

## Not covered

A fresh worktree still misses for every test that opens a file of the tree, since Go keys such a result on the file's absolute path and modification time.
Scoping verify to touched packages stays in note `verify-diff-scope`.
