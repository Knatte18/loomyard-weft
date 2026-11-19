```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds, find the test row that fails when the behaviour breaks.

**Per behaviour, the test that pins it**

1. `shippedOverNewer`: an untouched copy from a newer writer whose body differs from the default returns the shipped default and true. Pinned by `TestShippedOverNewer/UntouchedCopyFromANewerBinary` (`internal/stencilstore/skew_test.go:39`).
2. `shippedOverNewer`: a copy from an older writer is read as it stands. Pinned by `UntouchedCopyFromAnOlderBinary` (`skew_test.go:40`).
3. `shippedOverNewer`: a newer copy whose body equals the shipped default is read as it stands. Pinned by `NewerCopyWithTheShippedBody` (`skew_test.go:41`).
4. `shippedOverNewer`: an edited copy, where the `Classify` result is not `StateUntouched`, is read as it stands. Pinned by `EditedCopyFromANewerBinary` (`skew_test.go:42`).
5. `shippedOverNewer`: a running binary with a zero commit time never falls back. Pinned by `RunningBinaryWithNoCommitTime` (`skew_test.go:43`).
6. `shippedOverNewer`: a name the registry does not ship is read as it stands. Pinned by `StencilTheBinaryDoesNotShip` (`skew_test.go:44`).
7. `shippedOverNewer`: the boundary of `!recorded.Time.After(running.Time)` (equal commit time, or a copy with no recorded writer) and the `recorded.Revision == running.Revision` short-circuit. **None.**
8. `contentForRunning`: with `shipped == nil` it returns onDisk unchanged. Pinned indirectly by the existing `Read`/`ReadSet`/`ReadForEngine` tests (`TestReadForEngine`, `TestReadSet`), since `shipped` is nil in the stencilstore test binary.
9. `contentForRunning`: with a registered registry and a newer copy, it returns the default. **None.** No test sets `shipped` or `runningWriter`, and test binaries carry no VCS stamp, so the branch never runs.
10. `contentForRunning`: logs once per stencil through `newerCopyWarned`. **None.**
11. `Read` returns the shipped default for a skewed copy (`reconcile.go:39`). **None.**
12. `readInclude` keys an engine block's fallback on `EngineBlockName(engine, block)` and trims the default's final newline (`engineblock.go:164`). **None.**
13. `RegisterShipped` registers once and panics on a second call (`skew.go:29-34`). **None.**
14. The `contracts/stencils` `init` registers the embedded registry (`contracts/stencils/stencils.go:1140-1143`). **None.** The reach report lists contracts/stencils tests reaching `init`, but none asserts the registration.

**Findings**

- **MEDIUM**, class `design`
  - Location: `internal/stencilstore/skew.go:39-55`, `internal/stencilstore/reconcile.go:39`, `internal/stencilstore/engineblock.go:164`
  - Summary: the fallback wiring the diff adds has no covering test. That covers `contentForRunning` serving the shipped default when a registry is registered, its once-per-stencil Warn, `Read` returning the default, and `readInclude` keying engine blocks by `EngineBlockName` and trimming the default's newline.
  - Cause: `shipped` and `runningWriter` are package globals, and `runningWriter` reads `buildvcs.Running()`, which test binaries leave empty. No test can reach the branch without an injection seam. `_lym/landing/summary.md` states this ("is not exercised by a test").
  - Evidence: a grep of `*_test.go` for `RegisterShipped|contentForRunning|runningWriter|newerCopyWarned` finds only `skew_test.go:53`, which calls the pure `shippedOverNewer`.
  - Fix: make the registry and the running writer injectable without package globals, for example via `export_test.go` or a parameter. Then add a `Read`/`ReadSet` row over a `t.TempDir()` board that holds a newer-writer copy of a stencil and of an engine block, asserting the shipped bytes and a single log line.
- **LOW**, class `design`
  - Location: `internal/stencilstore/skew_test.go:38-45`
  - Summary: no row pins the boundary of the newer-than comparison at `skew.go:66`: an equal commit time, or a board copy with no recorded writer (zero time).
  - Effect: mutating `!recorded.Time.After(running.Time)` to `recorded.Time.Before(running.Time)` passes every row and would fall back on equal-time and writer-less copies. The doc at `skew.go:58` promises "a writer … with no recorded time is never newer".
  - Fix: add rows "CopyWithNoRecordedWriter" (`ApplyStamp` without `ApplyWriter`) and "CopyFromABinaryAtTheSameCommitTime", both expecting the copy as it stands.
- **LOW**, class `design`
  - Location: `internal/stencilstore/skew.go:28-34`
  - Summary: the `RegisterShipped` double-registration panic is an added behaviour with no test.
  - Evidence: the same grep found no `RegisterShipped` call in any test.
  - Fix: add a test that calls `RegisterShipped` on a nil-`shipped` state and expects a panic on the second call. This needs the injection seam from the MEDIUM finding, so the test does not mutate the shared global.

Tool calls:
1. Grepped `*_test.go` for `RegisterShipped`, `contentForRunning`, `runningWriter`, `newerCopyWarned`, `shippedOverNewer` and assignments to `shipped`. My context had only `skew_test.go` and could not show whether another test drives the wiring.
