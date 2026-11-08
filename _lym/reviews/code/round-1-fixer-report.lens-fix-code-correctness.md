```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix's code changes through the fix-code-correctness lens, running concrete inputs through each changed branch and test.

**Findings**

1. **LOW**: stale `awayPending` entry. This is `handle` in `inbound.go`, the `awayPending[orch] = true` / `case posted` / `case err == nil` block.
   - A turn whose first forward fails is kept in `awayPending` until a later outcome posts or finishes the same message.
   - If the retry never happens, the entry stays. One way this happens is the orch's channel being archived or dropped from the map, so the owed read is never made again.
   - A later operator message to that orch, sent while the presence is already slack, then posts and marks the orch in `awayFrom` although it turned nothing.
   - Effect: that orch's next slack notice is skipped at one wake.
   - Evidence: only the posted and handled-without-error paths delete the entry; no path clears it on a presence change or a removed channel.
   - I'd accept this as is, given how narrow the trigger is.

2. **LOW**: over-long comment line. This is the `forward` doc line `// A message with no text and no files, one over the body limit and one given up on after retryWindow are handled and not posted.`
   - It is about 126 characters after `//`, over the 120-character prose limit.
   - Split it at "and one given up on".

**Traces that held**
- `handle`:
  - A post that stands after a turn marks the orch and clears the pending entry.
  - A failed post leaves the entry and returns the error, so the cursor stays put.
  - The give-up, empty-body and oversize paths clear the entry and mark nothing.
  - If the presence write fails, nothing is pending, so nothing is marked.
- `forward`'s only caller is `handle`, so the new `(handled, posted, err)` result breaks no caller. The inner `handled, err :=` shadows the named results but compiles, and the gate passed.
- Each new test fails with its defect in place:
  - `FirstPostFailsAndTheRetryStands` fails under the old `turned && err == nil` code, because the retry sees `turned` as false.
  - `MessageWithNoTextAndNoFiles` and `MessageOverTheBodyLimit` fail under the old code, which marked the orch.
  - `MessageGivenUpOnDoesNotMarkALaterOne` fails if the `err == nil` delete is removed, because message 102 would then mark the orch.
  - The relay row for the terminal notice fails without the `presence != PresenceSlack` reset. Prime would be skipped, so only three notices arrive and `driveUntil` times out.
  - The relay row for the forgotten set fails if the set is taken after the early return. Prime would stay in the set and miss the later slack notice.
  - The first outbound row fails without `w.current = w.floor.get()` in `afterEmptyPoll`. The fourth wait would be `8*Floor` instead of `2*Floor`.
  - The raised-floor row fails if `>=` becomes `>`.

**Possible flake, not a defect**
- `OrchKeptByAWakeThatSentNoNoticeDoesNotSuppressALaterNotice` syncs on `beamReads >= reads+2`. If the outbound loop also calls the wrapped `Beams` and that call lands before the forwarding wake finishes, the later `setPresence(slack)` could race that wake's presence read.
- The outbound loop is parked on the manual clock in this test, so I think it makes no such call. I did not check this; I made no tool calls.

**Tool calls:** none. The context already held every changed file and test.
