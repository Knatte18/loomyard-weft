```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function or branch with a concrete case. I made no tool calls and judged from the inherited context.

**Per function or branch: case run, correct or not**
- `pause`, `slept` branch, `Sleep` returns nil: returns nil, goroutine already finished, no leak. Correct.
- `pause`, `slept` branch, `ctx` cancelled: `Sleep` returns `ctx.Err()`, `pause` returns it, `wait` maps it through `cancelErr`. Correct.
- `pause`, nil `changes` channel (watch failed, or the envkit and test fakes): the select blocks on the nil case, so only `slept` can fire. This is the old timer behavior. Correct.
- `pause`, `changes` fires, `wokeAt` 0.4s ago: `cancel()` ends the sleeping goroutine, `<-slept` drains the buffered channel, so no leak and no block. Then `rest` = 0.6s and a second `Sleep(ctx, rest)` runs on the parent context. Correct.
- `pause`, `changes` fires, `rest <= 0` (a long wake such as the gate, or a clock step): returns nil at once. Correct.
- `pause`, `changes` fires, `ctx` cancelled meanwhile: the drained `Sleep` returns a context error that is discarded. The follow-up `Sleep(ctx, rest)` returns `ctx.Err()`, or `rest <= 0` returns nil and the next `wake` hits `cancelErr`. Both paths return the right error. Correct.
- `pause`, `changes` and `slept` both ready: the select picks one at random. Both outcomes are valid (a full interval elapsed, so the early-wake path at worst adds a floor check). Correct.
- `wake`, `state.wokeAt = pr.Now()`: set at the start of every wake, before the first `pause`, so it is never zero when used. Correct.
- `wait`, `RecordChanges()` and `defer stopWatch()`: the watch opens after `Call` has written the record, so its directory exists. A write between the wake's record read and `pause` stays buffered (capacity 1) and ends the sleep at the floor. No lost wake. `stopWatch` runs before `markWait("")` and is never nil, because `watchFileChanges` always returns a function. Correct.
- `watchFileChanges`, `fswatch.Watch` error: logs a Warn, returns nil and a no-op stop. Correct.
- `watchFileChanges`, normal path: `stop` calls `watcher.Close()`, which closes `Events`, so the forwarding goroutine exits. `Close` is idempotent and `forward` selects on `done`, so no deadlock against the consumer. A Close error is logged at Debug. Correct.
- `watchFileChanges`, filter: the watch is filtered to the base name `pull-request.json`, so the `.lock` file and temp files do not wake it. Correct. The state write ends in a rename or create onto the target name, which is a delivered event.
- `openLandRequestWatch`, via `watchFileChanges`: same directory, same filter, same Warn level, same lifetime. The discarded `stop` keeps the old "lives until the process ends" behavior. Correct. Only the Warn text and fields changed: the message now reads "a directory could not be watched, so its waiter polls" and the fields gain `waiter`. No test or doc asserts the old text.
- `checkWiring`, new `RecordChanges` case: a nil seam is refused with the field name. Correct.
- The wait's own record writes (gate pass records `GatedHead`, mail failure records `MailedHead`, session registration) while `changes` is buffered: these are the only retrigger sources, and each writes once per head or episode, so the wait does not wake every floor while idle.

**Findings**
1. LOW, `design`, `internal/landingshed/prreview.go:402` and `:476`. A wake's own record write leaves a buffered event, so the next `pause` ends at the floor. The next wake is then an extra wake that polls nothing. `state.interval = nextPRWait(...)` also doubles on that wake, so the interval reaches the ceiling one step early after any gate or mail write. This is bounded, not a loop, and costs one cheap `readTree` wake. Evidence: `pause` selects `<-changes` with no drain of events the wake itself caused; `wake` sets `state.interval` at line 476 on every wake including early ones.

No other correctness defect found. On the focus question: a change does not reset the doubling `interval`. Early wakes still double it, so many record changes grow the interval while sleeping only the floor. After they stop, the next sleep is the already-doubled interval.

Tool calls: none; the manifest and diff already held everything this lens needed.
