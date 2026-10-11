# Card 1 — shuttle-await-poll

**Create:**
- `plan:internal/shuttleengine#AwaitAny` -> `func AwaitAny(ctx context.Context, paths []string, cap time.Duration) (string, bool, error)`
- `plan:internal/shuttleengine#awaitAny` -> `func awaitAny(ctx context.Context, clock Clock, paths []string, cap time.Duration) (string, bool, error)`
- `internal/shuttleengine/await_test.go`

**Edit:**
- `internal/burlercli#burlerCLI.awaitReviewCmd`
- `internal/burlercli#TestAwaitReviewVerb`

**Delete:**
- `internal/burlercli#awaitPollInterval`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `internal/shuttleengine#Clock`
- `internal/shuttleengine#pollFloor`
- `internal/burlercli#defaultAwaitCap`

**Intent:**
Move the capped file wait out of `lyx burler await-review` into one shared poll in `internal/shuttleengine`, in a new file `await.go`, so the `lyx shuttle await` verb of the next card and the burler verb run the same loop (decision record D9).

`awaitAny` is the loop: it stats every path in `paths` once per iteration, in order, and returns the first path that exists with `true`;
when none exists and `cap` has elapsed since the call began (measured by `clock.Now`) it returns `"", false, nil`;
a stat error other than not-exist returns it;
a cancelled `ctx` returns `ctx.Err()`.
Between iterations it sleeps `min(pollFloor, remaining)` through `clock.Sleep`, so the poll wakes at most once a second and never past the cap, meeting `PATTERN-daemon-wakeups` with no `time.NewTicker` and no sub-second literal.
It takes absolute paths and derives none (`PATTERN-told-geometry`), and watches no directory through `internal/fswatch`, since a watched path's parent may not exist yet.
`AwaitAny` is the exported entry over `realClock`; both CLIs call it.

`await_test.go` is a new untagged test file, since no existing test covers this surface: one table-driven test over a fake clock whose `Sleep` advances the time and can create a path on its n-th call, pinning: an existing path answers at once naming it; with several paths the one that appears first is named; the cap elapsing answers not ready with no error; a stat error other than not-exist is returned; a cancelled context is returned; and the sleep lengths never exceed one second or the remaining cap.
No test waits out a real interval (`PATTERN-test-speed`).

`burlerCLI.awaitReviewCmd` keeps its name, single argument, `--cap` default, `Short`, audience annotation and output keys (`ready`, `marker`, `message`) and calls `AwaitAny` with the one resolved path, so its poll moves from 200 ms to the one-second floor;
`awaitPollInterval` is deleted with it.
`TestAwaitReviewVerb` drops its "marker created while polling" row, which would now wait up to a second of real time; that behavior is pinned by `await_test.go` with the fake clock.
Its present-marker, absent-marker, missing-argument and stat-error rows stay as they are.

`docs/overview.md`'s burler line gains that `await-review` runs on shuttle's shared poll.

**ImpactSummary:** One caller of `awaitPollInterval`, in the same file; the burler verb's contract is unchanged, so only its own test row for the live-poll case moves.
