# Decision record — gate-fewer-new-binaries

## Goal

A verify on OSL-1046 links fewer test binaries Cortex XDR has never seen and re-runs fewer tests that already passed:
the unit tier no longer runs twice, a re-run on an unchanged tree takes the passed packages from Go's test cache, and a verify after a failure runs the failing test before the full command.
The numbers before and after go into `docs/benchmarks/gate-test-xdr.md`.

## Scope

- Verify runs only the `integration` tier.
  The hub's verify command lives in the board's `darn.yaml`, outside this worktree, so the parent changes it through `lyx config darn`; this worktree changes the repository text that names the two tiers, `CLAUDE.md`'s push rule.
- `lyx gate test` keeps per-invocation values out of the environment its tests read, so Go's test cache keys stay stable across runs on one tree.
- The loom verify gate (`internal/loomshed`), in both its round and whole-diff forms, runs a Publish failure record's failing tests before the told or derived command.
- The measurement on OSL-1046: the installed `lyx` with the two-tier verify before, a `.dev-bin` build of this branch with the integration-only verify after, each run twice on an unchanged tree.
- Out of scope: scoping verify to touched packages (note `verify-diff-scope`), and cache hits across worktrees for tests that open files of the tree, which Go keys on the file's absolute path.

## Decisions

### Verify runs the integration tier alone

Decision: the verify command becomes `lyx gate test --tags integration ./...`, and `CLAUDE.md`'s push rule names that one run.
Rationale: no test is excluded by the `integration` tag.
The only `//go:build !integration` files are TestMain shims whose `integration`-tagged TestMain does the same setup and more, so the tagged run already runs every untagged test.
A unit failure is reported by the tagged run too, so the separate unit run only adds its own wall time and its own set of test binaries.

### Strip the trace id from go test's environment

Decision: `lyx gate test` removes `LYX_TRACE_ID` from the environment it gives go test, beside the strand name it already removes.
Rationale: every `lyx` process mints or adopts a trace id and exports it, so each gate run hands go test a new value, and every test package whose code reads it through `internal/logger` misses the test cache on every run.
A run of `./internal/gateslot` with the variable pinned took its result from the cache on the second run, and without the pin missed every time.
Bound: a test process mints its own trace id instead of joining the gate run's; logging under go test is off unless `LYX_TRACE` is `1`, so only such an operator-requested trace loses the correlation.

### The prebuilt lyx gets a content-addressed path in the hub

Decision: inside a hub, a tagged `lyx gate test` builds `lyx` as now, then publishes it into a store directory under the hub's gate-slot directory, named by the binary's SHA-256, and exports that path; a binary already in the store is reused and its time refreshed.
Each prebuild prunes store entries other than its own whose time is older than a fixed retention.
Outside a hub the per-invocation path stays.
Rationale: the prebuilt path names a fresh per-invocation directory, so every package that reads it through `internal/testkit/lyxbin` misses the test cache on every tagged run, and those are among the slowest packages.
A path named by the binary's content is stable while the tree is unchanged, since the build runs with `-trimpath`, and changes whenever the binary does, so a cached result is never reused for a changed `lyx`.
Bound: a gate run whose binary is pruned while it still runs fails its remaining lyxbin tests with the stale-prebuilt error; the retention is far longer than a gate run, and every use refreshes the time.

### The failing tests run first

Decision: while a Publish failure record is present, the verify gate runs the record's failing tests, each by name in its package, before the told or derived command, and the `tmux` pass over packages after it.
Rationale: a failure that still holds is then known in seconds, and the full tiers run only once the failing test passes.

### Added after Discussion (parent, 2026-10-20): Loom plan verify also runs the integration tier alone

Decision: Verify runs only the integration tier also in the loom plan's ## verify: this branch changes pattern/PATTERN-test-speed.md's Webster gate/Publish/Finalize row and the internal/languageconfig doc's example verify_tiers to build, vet under each tag, then the integration tier alone.

Rationale: The board entry's verify-unit-tier-twice part names darn.yaml and the loom verify gates alike: the integration tier already runs every untagged test, and the operator approved dropping the separate unit tier on 2026-10-20.

## Constraints

- `PATTERN-gate-slot`: every gate build and test runs in a `-p`-capped, `-trimpath` hub slot; the prebuild stays inside the held slot.
- `PATTERN-lyx-temp-root`: the build output still goes to the process run directory of `internal/lyxtemp`; the store is a hub-level ephemeral directory under `.lyx`, never tracked (`PATTERN-durable-vs-ephemeral-state`).
- `PATTERN-lyxdirs-single-declarer`: the store path derives from the pool's directory, never a new `.lyx` literal.
- `PATTERN-spawn-observability`: no new process spawn; the existing build and go test spawns stay logged.
- The board's `darn.yaml` is hub-wide config on Bolt and is changed only by the parent.

## Auto-mode assumptions

- The parent changes the board's `darn.yaml` verify to the integration-only command; this branch changes no hub config.
- Module-wide `lyx gate test` runs are needed for the before and after measurement the board entry asks for, beyond the package-scoped runs used while coding.
- A retention of two hours covers every gate run.

## Open risks

- Other per-invocation inputs may still keep a few packages off the cache, such as `LYX_GATE_TEST_PID` and the slot variable that `internal/gatecli`'s own tests read; the measurement names the packages that still miss.
- A test that opens a file of the tree that the run itself rewrites, such as a log under the worktree, misses on every run; that is the test's to fix, not the gate's.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - store-race-error-paths-untested (class design, severity MEDIUM, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- A tagged `lyx gate test` exports a prebuilt path inside the hub's store, the same path for two runs on an unchanged tree, and a different one once `lyx` changes.
- A prebuild prunes a store entry past the retention and keeps a fresh one.
- go test's environment under `lyx gate test` carries no `LYX_TRACE_ID`.
- With a Publish failure record present, the gate's command runs the failing tests before the told or derived command, and the `tmux` package pass after it, in both forms.
- `docs/benchmarks/gate-test-xdr.md` records the before and after runs on OSL-1046, with the packages run and cached per run.
