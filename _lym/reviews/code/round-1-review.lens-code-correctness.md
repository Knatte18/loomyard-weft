```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the cross-hub mail diff, `9d2029216daaf759c6888dd9b26959d51f2b5923..HEAD`.

## List: function or branch, case run, verdict

- `Store.Deliver` phase 1 (`receive.go:124-165`):
  - Case run: a retry of a held message, an id clash, and a `readStrands` failure.
  - Result: a held message answers accepted, a clash answers refused, and the strand-list error surfaces only after both checks. Correct.
  - Case run: a reply naming an id that did not go to the sending hub. Refused before any confirm and no drop is journaled. Correct.
- `Store.Deliver` phase 3 (`receive.go:172-192`):
  - Case run: two overlapping deliveries, both confirmed yes. The second finds the first's stored message and answers its id. Correct.
  - Case run: confirm `no`. The drop is journaled and the answer is refused. Correct.
  - Case run: confirmed and the id clashes. Refused. Correct.
  - Case run: a transient confirm error. It returns an error with nothing stored or dropped, so the sender retries. Correct.
  - The store lock is not held across the confirm. Correct.
- `Store.Confirm` (`receive.go:110-119`): Case run: a message that is delivered, returned, absent or body-altered. It answers `no` for each, and the post time compares with `Equal`. Correct. The journal round-trips the nanosecond time (`journal.go:258` sets `At` in UTC, which strips the monotonic reading).
- `trustedSender` (`receive.go:271-282`):
  - Case run: `orch-2`, a non-Linux platform, a config load failure, and `<sn>:<slug>:orch`.
  - Result: `agentname.Parse` keeps `orch-2` as a role that is not `orch`, so it is untrusted. The other three cases follow the spec. Correct.
- `noticeDropLocked` (`receive.go:319-348`):
  - Case run: five drops, a sixth, then a drop after the hour.
  - Result: notices 1 to 5, with the fifth carrying the "journaled only" text. The sixth sends none. The next hour opens a window carrying the undisclosed count. Correct, and the map is keyed on the sending hub path.
  - See finding 1 below.
- `quoteAnswer` (`outbound.go:52-67`):
  - Case run: a multibyte rune straddling byte 200, a backslash, a quote, a newline and invalid UTF-8.
  - Result: control characters become spaces, `strings.Map` turns invalid bytes into U+FFFD, the cut backs up to a rune start, and the escapes are applied after the cut. Correct.
- `outboundWorker.run`, `conclude` and `attempt` (`outbound.go:230-366`):
  - Case run: refused, accepted, a transient answer before the hour, a transient answer at the hour, an accepted answer at the hour, and a failed journal write. Each gives the specified outcome. Correct.
  - Case run: the wait cap near the deadline. `min(wait, max(remaining, base))` stays bounded. Correct.
  - Case run: the start cap. It counts only successful spawns, resets on any answered call including an unknown-verb refusal, and `spawn == nil` gives `connLostError` and `codeStartCapped` (`client.go:84-89`). Correct.
  - Case run: a return crossing an in-flight deliver. The worker is sequential, so it cannot. Correct.
  - See finding 2 below.
- `retire` and `wakeOutbound` (`outbound.go:174-187, 273-282`):
  - Case run: a message queued while the worker retires.
  - Result: `retire` re-checks pending under the scheduler lock, and `wakeOutbound` starts a new worker when the old one is gone. The lock order is scheduler then store, with no inversion. Correct.
- `settleReturned` and `settleDelivered` (`outbound.go:121-149`): Case run: the message is no longer pending, or the sender is not a listed strand. Each is a no-op or skips the notice. Correct.
- `postOtherHub` (`crosshub.go:51-86`): Case run: a program sender, a program receiver, an unresolved shortname, an over-cap body, and `Wait`. Each is refused or handled as D2 says, and the wait is registered inside the same `do`. Correct.
- `Reply` for mail from another hub (`verbs.go:754-768`): Case run: an over-cap reply, and a reply to a chase. A chase is first mapped to the original, an oversize reply is refused, and `Answers` plus `markAnswered` stop the original's chase. Correct.
- Fold (`fold.go`: `applyOutbound`, `applyInbound`, `applyCarried`, `carryRecords`, `prune`):
  - Case run: day rollover with a pending outbound, a delivered outbound, and an inbound carried message.
  - Result: pending outbound is carried with `OrigAt` and `moveOrigins`, delivered is dropped, and inbound carries as `of: inbound` with its stamp fields. Correct.
- `settledOutcomeLocked` (`pool.go:431-446`): Case run: a late waiter on a returned outbound, and one on an outbound that already has a reply. Both give the right outcome. Correct.
- `releaseGoneHolder` (`sweep.go:230`): Case run: an inbound message with an empty path and a gone holder. It is journaled undelivered. Correct.
- `noticeUnansweredLocked` (`chase.go:247-257`, unchanged):
  - Case run: a settle of a cross-hub message.
  - Result: `findStrand` over this hub's listing never lists the other hub's name, so no notice goes out. Correct.
- `hubregistry.Record`, `Lookup` and `Live`:
  - Case run: a missing path, a changed shortname, an absent Bolt record, and a duplicate shortname.
  - Result: gone entries drop, the others are kept, and the lookup refuses on none or several with the way forward. Correct.
- `hubDirectory.TrustedHubs`, `Lookup` and `Record`, and `daemon.recordHub`: Case run: an unreadable state directory, and a record failure. A named refusal is returned, one warning is logged per episode, and the daemon serves on. Correct.
- `hubsCmd`: it never connects to a daemon, because `runVerb` builds a client without dialling (`cli.go:167-177`). Correct.
- `MessageLine`, `composeReminder` and `messageFields`: Case run: a local message and a cross-hub one. The local line is unchanged, and the cross-hub line has the stamp after the sender's quote with `from` kept bare. Correct.
- `isNotice` (`reminder.go:28`): a strand of another hub is not a program, so it is agent mail and a pane-mail strand is reminded, not typed. Correct.

## Findings

1. **LOW, design.** `noticeDropLocked` (`receive.go:331-347`)
   - It increments `window.notices` and clears `undisclosed` before `postDropNoticeLocked` runs.
   - If that post fails (for example, no listed orch), the window is spent and the lost notice is not counted into a later one.
   - The drop stays in the journal, so nothing is lost silently, but "every drop is noticed or follows a notice" can fail when the post errors.
   - Fix: count the notice only after the post succeeds, or keep the failed notice's drops in `undisclosed`.

2. **LOW, design.** `outboundWorker.attempt` (`outbound.go:334`)
   - `Call` takes no stop signal, and the worker is in the scheduler's `wg`.
   - `Scheduler.Stop` (`deliver.go:76`, which waits for sends in flight) can therefore wait up to the 30 s call timeout.
   - A replaced binary (`server.go` poll) can then keep its lock for that long, and verbs sent meanwhile hit a daemon that is shutting down.
   - Bounded, and arguably the specified "in flight ends" behaviour, but D2 does not name this shutdown delay.

## Tool calls

1. `grep` over `eventUnanswered`, `settleUnansweredLocked` and `isNotice`: the settle and notice code was not in the diff, and I needed to check that a cross-hub settle sends no notice.
2. `grep` and `sed` over `agentname.Parse` and `Reply`: I needed role parsing for `orch-N`, and the `Reply` lines before the hunk.
3. `sed` of `Client.connect` and `journalWriter.append`: I needed the nil-spawn behaviour, and to confirm `At` is set and the pool is applied on append.
4. `grep` of `runVerb`: to check that `lym creel hubs` starts no daemon.
