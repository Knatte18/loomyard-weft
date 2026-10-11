# Card 1 — shuttle-await-poll

**Create:**
- `internal/shuttleengine#AwaitAny`
- `internal/shuttleengine#awaitAny`
- `internal/shuttleengine#DefaultAwaitCap`
- `internal/shuttleengine/await_test.go`

**Edit:**
- `internal/burlercli#burlerCLI.awaitReviewCmd`
- `internal/burlercli#TestAwaitReviewVerb`
- `internal/shuttleengine/doc.go`

**Delete:**
- `internal/burlercli#awaitPollInterval`
- `internal/burlercli#defaultAwaitCap`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `internal/shuttleengine#Clock`
- `internal/shuttleengine#pollFloor`

**Intent:**
Move the capped file wait out of `lyx burler await-review` into one shared poll in `internal/shuttleengine`, in a new file `await.go`, so the `lyx shuttle await` verb of the next card and the burler verb run the same loop (decision record D9).

`awaitAny` is the loop: it stats every path in `paths` once per iteration, in order, and returns the first path that exists with `true`;
when none exists and `cap` has elapsed since the call began (measured by `clock.Now`) it returns `"", false, nil`;
a stat error other than not-exist returns it;
a cancelled `ctx` returns `ctx.Err()`.
Between iterations it sleeps `min(pollFloor, remaining)` through `clock.Sleep`, so the poll wakes at most once a second and never past the cap, meeting `PATTERN-daemon-wakeups` with no `time.NewTicker` and no sub-second literal.
It takes absolute paths and derives none (`PATTERN-told-geometry`), and watches no directory through `internal/fswatch`, since a watched path's parent may not exist yet.
`AwaitAny` is the exported entry over `realClock`; both CLIs call it.
`DefaultAwaitCap`, declared beside it, is the one default cap both verbs' `--cap` flags take: under the 5-minute prompt-cache lifetime, so a caller that runs the wait with a Bash tool timeout above the cap (300000 ms) and re-runs it keeps its cache warm.

`doc.go` gains a short paragraph on the shared wait: `AwaitAny` stats the told paths once a second until one exists or the cap elapses, is the poll every capped keep-cache-warm wait verb runs on, and watches no directory.

`await_test.go` is a new untagged test file, since no existing test covers this surface: one table-driven test over a fake clock whose `Sleep` advances the time and can create a path on its n-th call, pinning: an existing path answers at once naming it; with several paths the one that appears first is named; the cap elapsing answers not ready with no error; a stat error other than not-exist is returned; a cancelled context is returned; and the sleep lengths never exceed one second or the remaining cap.
No test waits out a real interval (`PATTERN-test-speed`).

`burlerCLI.awaitReviewCmd` keeps its name, single argument, `--cap` default value, `Short`, audience annotation and output keys (`ready`, `marker`, `message`) and calls `AwaitAny` with the one resolved path, so its poll moves from 200 ms to the one-second floor;
its `--cap` flag defaults to `shuttleengine.DefaultAwaitCap`, and `awaitPollInterval` and `defaultAwaitCap` are deleted with the loop.
`TestAwaitReviewVerb` drops its "marker created while polling" row, which would now wait up to a second of real time; that behavior is pinned by `await_test.go` with the fake clock.
Its present-marker, absent-marker, missing-argument and stat-error rows stay as they are.

`docs/overview.md`'s burler line gains that `await-review` runs on shuttle's shared poll.

**ImpactSummary:** `awaitPollInterval` and `defaultAwaitCap` each have one reference, in `awaitReviewCmd` on this card; the burler verb's contract is unchanged, so only its own test row for the live-poll case moves.
