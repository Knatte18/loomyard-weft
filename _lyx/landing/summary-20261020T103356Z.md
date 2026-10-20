# Verify runs the integration tier once and reuses passed results

A verify now links fewer test binaries that Cortex XDR has never seen, and a re-run on an unchanged tree takes passed packages from Go's test cache.
Before, verify ran `lyx gate test ./... && lyx gate test --tags integration ./...`.
The `integration` tag compiles and runs every untagged test too, so the unit tier ran twice and each tier linked its own test binaries, which XDR holds 8-10 s each on OSL-1046.

## What changes

- **Verify runs the `integration` tier alone.**
  No test file is excluded by the tag: the only `!integration` files are TestMain shims whose tagged TestMain does the same setup and more.
  Changed to match:
  - The push rule in `CLAUDE.md`.
  - The Webster gate/Publish/Finalize row and the Publish-failure row of `pattern/PATTERN-test-speed.md`.
  - The example `verify_tiers` in the `internal/languageconfig` doc.
  - The impacted-set command of `internal/impactset`, which the Webster-Burler-Fix round gate runs: it now emits build, vet under each tag, then `go test -tags integration` over the set.
- **`lyx gate test` strips `LYX_TRACE_ID` from go test's environment.**
  Every `lyx` process mints and exports a trace id, so each gate run gave go test a new value, and every package that reads it through `internal/logger` missed the test cache on every run.
  `logger.TraceIDEnv` names the variable, and the existing literals in `internal/reedengine` and `internal/shedverbs` use it.
- **A tagged run inside a hub exports a content-addressed prebuilt `lyx`.**
  The binary is published into `Pool.PrebuiltDir()` (a `prebuilt` directory in the gate-slot directory) as `lyx-<first 16 hex of its SHA-256>`.
  Two runs on an unchanged tree export the same path, so packages that read it through `internal/testkit/lyxbin` stay cached, and a changed `lyx` gets a new path.
  A binary already stored is reused and its modification time refreshed.
  Each prebuild prunes other entries last used more than two hours ago.
  Outside a hub the per-invocation path stays.
- **A verify after a Publish failure runs the failing tests first.**
  In `internal/loomshed`, both forms of the verify gate (round and whole-diff) now put the record's failing tests, each by name in its package, before the told or derived command.
  The `tmux` pass over the impacted packages stays after it.
  A failure that still holds is known in seconds, and the full tier runs only once it passes.
- **`internal/shedbuild`'s `TestLoad_RelativePathErrorsBeforeAnyRead` no longer writes a fixture into the source tree.**
  It names one of the package's own files instead, since the temporary file changed a directory listing that the source scans of other packages read, and Go keys their cached result on it.
- `docs/benchmarks/gate-test-xdr.md` records the before and after runs on OSL-1046.
  An unchanged-tree re-run of the `integration` tier ran 0 of 162 packages and took 26.6 s, against 186.0 s for the two-tier verify before.

## Design decisions

- **Content-addressed store instead of a stable fixed path.**
  A fixed path would let a cached result outlive a changed `lyx`; a path named by the binary's content changes exactly when the binary does.
  This relies on the build running with `-trimpath`, which makes an unchanged tree build the same bytes.
- **Retention bound.**
  A gate run whose binary is pruned while it still runs fails its remaining `lyxbin` tests with the stale-prebuilt error.
  Two hours is far longer than a gate run, and every use refreshes the time.
  `refreshStored` re-stats the entry after the refresh and republishes it if a concurrent prune removed it.
- **Trace correlation bound.**
  A test process under `lyx gate test` mints its own trace id instead of joining the gate run's.
  Logging under go test is off unless `LYX_TRACE` is `1`, so only an operator-requested trace loses the correlation.
- **Failing tests first is a prefix to the command, not a separate gate.**
  It assumes the told command is a single `&&` chain, which the loom plan parser guarantees for verify lines.
- **The store path derives from the pool's directory** and adds no `.lyx` literal (`PATTERN-lyxdirs-single-declarer`); the prebuild stays inside the held slot.
- The new cross-cutting rules, a content-stable exported prebuilt path and no per-invocation value in go test's environment, are stated in `pattern/PATTERN-test-speed.md`.

Out of scope: scoping verify to touched packages, and cache hits across worktrees for tests that open files of the tree, which Go keys on the file's absolute path.

## To check or run by hand

- The hub's verify command lives in the board's `darn.yaml`, outside this change.
  Set it to `lyx gate test --tags integration ./...` through `lyx config darn`; until then the hub still runs both tiers.
- The full `lyx gate test --tags integration ./...` on a clean tree was not re-run after the last fixes; only the touched packages were.
- The `tmux` pass of a Publish verify failed once on `internal/reedcli`'s `TestWatchdogReap/NeverEnteredOrphanIsStillReaped`, whose 1.8 s wait for the reap is too tight on a loaded host.
  This change leaves that test alone, and it passes when re-run on its own; the flake is issue #769.
- Other per-invocation inputs may still keep a few packages off the cache, such as `LYX_GATE_TEST_PID` and the slot variable that `internal/gatecli`'s own tests read.
- `mill-config.yaml:123` `done_gate` still runs the untagged tier before the integration tier; it is the retired mill tool's config and was left alone.
- Check by hand:
  - `internal/impactset/derive.go:200`: the impacted-set command emits only the `integration` test step after the vet steps.
  - `internal/gatecli/prebuild.go:127`: a prune that removes an entry just after another run refreshed it makes that run republish the entry.
    The residual window, a prune that read the old modification time before the refresh and removes it after the re-stat, remains.
  - `internal/gatecli/prebuild_test.go:1`: the republish branch has rows for a missing entry and for an existing one.
  - `internal/loomshed/verifygate_integration_test.go:249`: a plan-verify failure record with failing tests and no package pass assembles the failing tests, then the derived command, with nothing after it.
  - `pattern/PATTERN-test-speed.md:37`: the sentence on the content-named path and the stripped trace id reads correctly.
  - `CLAUDE.md:37`: the push-rule bullet breaks at each clause.
