# Take real time out of test waits; narrowed gates skip unwatched tests

Tier 1 tests spent seconds on fixed sleeps, real deadlines and builds that repeat on every pass.
This change takes the real time out of those waits through injected seams or by waiting on the event itself, never by a shorter bound.
It also adds a `//lym:watch` marker so a narrowed gate runs a build test only when a file it guards changes.

## Real-time waits

- `internal/gitexec`: `TestRun_RemoteBounds` waited out a real 1 s remote deadline in four rows and ran a local `sleep 2` alias.
  The remote deadline now goes through `armDeadline`, a seam beside the existing stall-timer seam.
  `SetDeadlineForTest` ends an attempt once the ext:: script has started, and the reported `Timeout` is still the configured deadline.
  The local row asserts that a local command arms no deadline, instead of outliving one.
- `internal/termwindow`: `attachLineWaiting` takes a poll count and interval.
  Production keeps 30 polls of one second, and the removed-worktree row polls every 50 ms within the same 3 s bound.
- `internal/verifyrun`: the cancellation test cancels once the child prints, not after a fixed sleep.
  The expiry test expires its context once the backgrounded descendant has written its pid file, not after a fixed 1 s.
- `internal/gatecli`: the detached-run stand-ins and card lines that slept 4 to 5 s now wait for a release file the test creates.
  The front's lock and claim poll and the runner's lease read are `detachSeams` fields, one second in production,
  so the tests set them in milliseconds through the helper's environment.
- `internal/testkit/lymbin`: `BuildInto` creates its output directory before building.
  `TestBuildInto_UnwritableDirReturnsError` now fails on that directory's `*fs.PathError` without running `go build`.

## Watched tests

A top-level test marked `//lym:watch <pattern>...` guards only the files its patterns name.
`path.Match` matches a pattern holding a `/` against the worktree-relative path, and any other against the base name.
Under the `go` scoper, a narrowed computed or `--base` share and a `--card` scope skip such a test,
unless a changed file matches one of its patterns or its own file changed.

- `impactset.FindWatchedTests` parses the marker, with the same placement rules and `*GuardScanError` kinds as the guard and seam markers,
  plus `WatchMalformed` for a marker with no pattern or a malformed one.
  `DeriveTargets` fills `Targets.Unwatched` for packages in the set and drops unwatched tests from `Guards`.
- A `--card` scope reads its changes as the diff from the merge base with the parent branch.
  With no parent branch, or an unreadable diff, it skips nothing.
- A skipping tier gets `-skip '^(<names>)$'` after the words from `--`, and a stderr line names the skipped tests.
  The skip is left off when the caller's words carry a filter flag, or when the tier command holds no `{args}`.
- The empty-tier check does not count a skipped test.
  In the green record, a whole pass with empty args covers a skipping run, and a skipping run never covers a whole one.
- A whole gate skips nothing, so `TestCrossCompileLinux` and `TestCSharpFixtures_CompileWithDotnet` still run in every whole pass.
  `TestCrossCompileLinux` watches `*_linux.go` and `*_unix.go`, and `TestCSharpFixtures_CompileWithDotnet` watches `*.cs`.

`PATTERN-test-speed` gains a section for the marker, and `PATTERN.md`'s entry names it.

## Design decisions

- No bound was shortened.
  Waits left as they were include the bounds that cover a process start, such as the keep-warm caps, claim bounds and pid-file waits.
  So are the rows whose assertion is that time passed, such as a call outlasting a short cap.
  The termwindow rows that let the full wait run out, so the shell is kept, also keep their bound.
- The watch skip is keyed by test name, because `go test -skip` cannot name a package.
  A test with the same name in another package of the same run is skipped too; `PATTERN-test-speed` states this bound.
- The card scope uses the branch diff rather than the card's own files.
  The card's file list names directories and refs, not changed paths, and the branch diff never skips a test a change could reach.
