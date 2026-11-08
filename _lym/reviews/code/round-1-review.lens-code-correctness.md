```yaml
lens: code-correctness
outcome: reported
```

Task: review the correctness of the changed code (lens code-correctness).

**List (function or branch, case run, verdict)**
- `inbound.handle`, write `in.awayFrom[orch] = true`: correct. `handle` is reached only through `attempt` → `readChannel`/`readThread`, called from `drain` and `retryOwed`, and both hold `in.mu` (`inbound.go:285`, `:322`). `takeAwayFrom` locks `in.mu` (`:160`), so the map is never touched concurrently.
- `handle`, forward fails with a Post error, first try: the presence is turned and `forward` returns `(false, err)`, so the orch is not marked. Retry: `awayOnOperatorMessage` now reports `turned=false` because the presence is already slack, so the orch is never marked even when the forward succeeds. Not correct, see F1.
- `handle`, `forward` returns `(true, nil)` without posting: not correct, see F2. The three cases are an oversize message (`inbound.go:705-708`), a give-up after `retryWindow` (`:736-739`), and an empty body (`:701-704`).
- `awayOnOperatorMessage`, WritePresence error: returns false and logs, so no skip. Correct. Presence already slack: returns false. Correct.
- `takeAwayFrom`, empty set: returns the empty map and installs a fresh one. Correct.
- `noticePresenceChange`, `presence == w.noticed`: the set is taken and cleared, then it returns. Correct, since no notice is due.
- `noticePresenceChange`, first wake (`!known`): the set is cleared and nothing is mailed. Correct.
- `noticePresenceChange`, change back to terminal: `awayFrom = nil`, so every orch is told. Correct.
- `noticePresenceChange`, a notice to the skipped orch fails: skipping happens before the post, so it is not counted as a failure. Correct.
- `noticePresenceChange`, `ReadPresence` fails: the function is not called and the set keeps accumulating. Correct.
- `noticePresenceChange`, a beam orch not yet in `beams`: it gets no notice from `noticePresence` anyway, so nothing is lost. Correct.
- `noticePresenceChange`, turn inside `retryOwed`: `wake` runs `retryOwed` after the notice, so the orch stays in the set and is skipped at the next wake. Correct.
- `noticePresence` skip, key match: `skip[receiver]` uses the name from `orchName(shortname, slug)`, and the channel map's orch key is the same name. The new relay test exercises the prime orch. Correct.
- `afterEmptyPoll`, `elapsed >= floor`: returns 0 and `current` is set to the floor. Correct. With the floor raised to 10s by a rate limit, a 7s poll falls to `afterIdle` and backs off. Correct. Reached only on `ErrWaitTimeout && !handlerRan`. `runOutbound` then calls `waitFor(…, 0)`, with `afterRead` between reads, so it cannot spin faster than the poll lasts. Correct.
- `afterEmptyPoll`, boundary: `elapsed == floor` returns 0, with no hot loop since each poll takes at least the floor. Correct.
- `read`, injected clock: `pollStarted` and the later `Now()` both use `out.clock`, so the elapsed time is the clock's. Correct.
- `creelStopHookCommand`, three shapes. Correct for each:
  - Empty `turnHookCommand`: the string equals the old `in=$(cat); delivery; record`.
  - Non-empty: `in=$(cat); step; delivery; record`. `$in` is read once and the step's status is ignored.
  - Mail handed out: `exit 0` after the step has run, with no record.
  - No mail: the exit status is the record's, matching the doc. The dropped `s=$?; … exit $s` is now unneeded because the record is last.
- `creelOpenStopHookCommand`: `in=$(cat);` moved from the delivery step into the command. The string is unchanged. Correct.
- `turnHookStep` before delivery: its subshell `cd` does not leak, and its output is discarded. Correct.

**Findings**

F1. LOW, design, `internal/slackengine/inbound.go:646-651` (with `:661-671`).
- On a retried forward, the orch is never marked, so it gets the `[slack presence]` notice that wakes it a second time.
- The first attempt turns the presence and the Post fails. The retry sees the presence already slack, so `turned=false`.
- The result is the wasted wake the change set out to remove, only in this failure path.
- Evidence: `handle` at `:646-651`, `awayOnOperatorMessage` at `:661-671`, `retryOrGiveUp` at `:742`.

F2. LOW, design, `internal/slackengine/inbound.go:648-651`.
- `turned && err == nil` marks the orch even when `forward` posted nothing: an oversize message (answered in the channel), a give-up after `retryWindow`, or a body with no text and no files.
- The orch was not woken, yet its presence notice is skipped, and the orch never learns the presence went to slack.
- The doc comment "the forward stood" is not what the condition tests.
- Evidence: `forward` returns `(true, nil)` without a Post at `inbound.go:701-708` and `:736-739`.
- Fix direction: mark the orch only after a successful Post, for example by having `forward` report whether it posted.

**Tool calls (2)**
1. Read `inbound.go:695-749` for `forward`, `retryOrGiveUp` and `settle`; the manifest lacked them.
2. grep in `relay.go` for `runOutbound`, to check how a wait of 0 is handled; the manifest lacked it.
