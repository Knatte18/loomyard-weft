# Take real-time waits and repeated builds out of tier 1 tests

Tier 1 tests waited out real deadlines, sleeps and leases, and rebuilt the same binary on every pass.
This change removes that time without shortening any bound that covers a process or binary start.
On OSL-1046, Cortex XDR holds each new binary about 10 s on its first start, so those bounds keep their length.
Items 7 and 9 of `docs/research/speed/tier1-test-revision-2026-11-06/report.md` are the source.

## What changes

- `internal/gitexec`: `TestRun_RemoteBounds` no longer sits out the remote deadline or a literal `sleep 2`.
  A new `armDeadline` hook in `gitexec.go` builds each remote attempt's context.
  The test replaces it, ends the attempt once its remote is ready, and records the deadlines armed.
  A new test pins the production deadline.
- `internal/gatecli`: the detached-run front and runner take their poll period and lease read period from `detachSeams` (`pollPeriod`, `leaseReadPeriod`).
  Production passes `frontPollPeriod` and `leaseReadPeriod`, and a zero value falls back to them.
  `TestGateTest_DetachedRun` and the scope tests use short periods instead of 2 to 5 s stand-in delays and 3 s leases.
- `internal/termwindow`: `attachLineWaiting` takes a poll count and a poll interval, and `attachLine` passes 30 polls of one second.
  The 3 s test row now uses a short interval.
- `internal/verifyrun`: the 1 s expiry tests wait on the event instead of a fixed sleep, and the sleep starts before the first write.
- `internal/testkit/lymbin`: `compile` creates the output directory before it builds.
  `TestBuildInto_UnwritableDirReturnsError` now fails before the build, which saves the 2.3 s binary build.
- New `//lym:watch <pattern>...` marker, parsed by `internal/impactset/watches.go`:
  - A marked test runs in a narrowed gate only when a changed file matches a pattern or the test's own file changed.
  - A whole gate always runs it.
  - A diff-scoped key narrows over its diff, and a `card` key over the diff from the merge base with the parent branch.
    With no parent branch or an unreadable diff, a `card` run skips nothing.
  - The skipped tests reach `go test` as `-skip '^(<names>)$'` after the caller's own args, with a line on stderr naming them.
    No skip is added when the caller's args carry a filter flag or the tier command has no `{args}`.
  - The guard step drops a watched guard test that no changed file concerns.
  - A misplaced or malformed marker refuses the gate with its file, line and way forward.
- `TestCrossCompileLinux` carries `//lym:watch *_linux.go *_unix.go`, and the dotnet fixture build carries its own marker.
  Both stay in every whole pass.
- `PATTERN.md` and `pattern/PATTERN-test-speed.md` document the marker.

## Design decisions

- Every wait is removed by an injected period or by waiting on the event, never by a shorter bound.
- `armDeadline` is a package-level variable rather than a seam field, so the tests that replace it run serially.
  `PATTERN-test-speed` names this as an exception for `internal/gitexec`'s remote-attempt bounds.
- The `-skip` filter matches by test name.
  A test of the same name in another package of the run is skipped too, and the pattern notes this bound.

## To check

- `armDeadline` in `internal/gitexec/gitexec.go` is a package-level variable, which departs from "Time is injectable".
  Decide whether the documented exception is acceptable or the hook should move into a parameter.
- The `card` scope's watch skip (`computeScope` and `cardChanges` in `internal/gatecli/scope.go`) has no test that runs it end to end.
- The guard step's drop of a watched guard test (`scanGuards` in `internal/gatecli/scope.go`) has no test that runs it end to end.
- Run `lym gate test` over a whole pass once, to confirm `TestCrossCompileLinux` and the dotnet build still run there and the timings fall.
