```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens over cards 1 to 10. For each guard, I say whether it is reachable and whether the guarded site recurs unguarded.

**Per guard: reachable or not**

Card 1
- `Lookup` none and several refusals: reachable. Rows exist.
- Corrupt record refusal: reachable. `state` wraps `ErrDecode` (`internal/state/state.go`, `readJSONUnlocked`), and `UpdateJSON` aborts before mutate.
- ENOTDIR entry kept: reachable.
- Absent directory: no guard needed, since `UpdateJSON` creates the parents and `ReadJSON` tolerates an absent lock directory.
- The way forward in the none and corrupt texts is not always true. See F3.

Card 2
- Local layer row and malformed YAML: reachable.

Card 3
- Unknown `Of` default branch and carried-beside-original: reachable by hand-written journal lines.

Card 4
- `ErrDaemonRunning` records nothing: reachable.
- Failing record still serves: reachable.
- `registryErr` branch: reachable, because `UserStateDir` fails with no `HOME`.
- `hubRecordInterval` skip: reachable.

Card 5
- Corrupt record, invalid `creel.yaml` and standalone refusal: reachable.

Card 6
- `refuseOtherHub` at `inject`, `Interrupt`, `Hold`, `Release` and `post --interrupt`: reachable. Dispatch passes `req.Receiver` straight through (`server.go`, dispatch), and `InjectCommand` lands in `inject`, so no unguarded site recurs. `Withdraw` is program-only, outside D8, and touches only local entries.
- `postOtherHub` refusals 1 to 4: reachable, in order. The local `validateLine` runs first and rejects only what also fails the stamped measure.
- `WaitReply` rows: reachable through `reopen`.

Card 7
- Step 1 and step 2 refusals, no-`Mail`, id clash at step 3 and step 6, unknown receiver after the duplicate check, confirm failed or neither word, and `no` with a held message: all reachable. An overlapping retry or a hand replay reaches them.
- Nil seam: fail-loud wiring guard, exempt.
- Journal-failure rows: reachable through `FailJournalWrites`.
- `PanePID` 0 row: reachable.
- Drop reached with a differing reply-to: reachable at card 7, unreachable after card 9. See F1.

Card 8
- All transient codes: reachable.
- Start cap and hour return: reachable.
- In-flight-across-the-hour row: reachable through a journal line posted just under an hour earlier. On the live retry ladder the last attempt before the hour falls more than 30 seconds out, so the row needs that line.
- Text quoting: `daemonRefusal` is reachable, but the same site recurs on two paths the card does not name and no row can reach. See F2.

Card 9
- `Store.Deliver` reply guard: reachable through a hand-written `deliver` from a resolved hub.
- Forward refusal: reachable while the message is unread.
- `releaseGoneHolder` branch: reachable. `isStrandName` is true for another hub's name.
- Chase and settle: no notice goes out, reachable. `noticeUnansweredLocked` finds the sender unlisted.
- Mail to a fixed orch name with no live strand passes `checkReceiverLocked`, then is swept undelivered. This is what D4 prescribes, so it is no finding.
- `Reply`'s holder guard keeps the sender a strand, since forward and step 1 keep programs from holding such mail. No site recurs.

Card 10
- Adds no guard.

**Findings**

F1. MEDIUM, consistency
- Location: card 7 row "A `deliver` of a real pending id differing in each of sender, receiver, reply-to, mark and post time is refused as a drop" (`_lym/plan/07-receive-and-confirm.md:94`), against card 9's `Store.Deliver` guard (`_lym/plan/09-replies-across-hubs.md:37-38`).
- Summary: from card 9 on, the reply-to case of this row can no longer reach the confirm or the drop, and no card updates the row.
- Evidence:
  - Card 7 rows post on the sending rig, so the pending message is plain mail with `ReplyTo` 0 (`07-receive-and-confirm.md:91`), and the differing delivery carries a non-zero `ReplyTo`.
  - Card 9 runs "after the duplicate check of step 3 and with its receiver rules", before the confirm, and refuses any `ReplyTo` with no matching `outbound` entry, with a reason naming the id and no drop.
  - So the row's assertion of a drop and a notice fails at card 9's gate.
  - Card 9's Edit list (`09-replies-across-hubs.md:3-17`) does not name `TestDaemon_CrossHubDeliver`.
- Fix: card 9 lists `plan:internal/creelengine_test#TestDaemon_CrossHubDeliver` and changes the row so its `ReplyTo` names a real `outbound` entry of the receiving hub, sent to the sending hub by the delivery's receiver. That passes the guard, and the confirm then answers no. Otherwise the row must assert the reply refusal instead of a drop.

F2. MEDIUM, design
- Location: card 8 (`_lym/plan/08-outbound-worker.md:64-65`), `internal/creelengine#outboundWorker.attempt`, `Client.exchange`. Rows: `08-outbound-worker.md:93-95`, with `listenHubFake` at `07-receive-and-confirm.md:89`.
- Summary: D2's bound of 200 escaped bytes in a `CREEL` notice is guarded only on the plain refusal path. The same socket-written text reaches the worker on two other paths, and the card names no code or quoting for them.
- Evidence:
  - `internal/creelengine/client.go:244-246` puts `resp.HubPath`, which the socket wrote and which is unbounded, into an `fmt.Errorf`. This is the first branch any binder of the socket hits, since it need not echo the hub path.
  - `client.go:248-253` returns `resp.Error` as `codedRefusal` when the socket sets a code. That path bypasses the `errors.New` branch card 8 turns into `daemonRefusal`.
  - Neither error is `daemonRefusal`, `connLostError` or `ErrWaitTimeout`, so card 8's mapping gives them no code. Both become the "last transient answer" that D2 places in a notice from `&lt;shortname&gt;:CREEL`, whose mail an agent follows as the operator's own.
  - The sentence "whatever the socket that wrote it" states the intent, but no mechanism or row carries it.
  - `listenHubFake` always writes the rig's hub path, so no planned row can reach the mismatch branch.
- Fix:
  - Card 8 states that every other `Client.exchange` failure either takes a code creel writes, such as `bad-answer`, or passes through `quoteAnswer`.
  - It treats a coded refusal as `daemonRefusal`.
  - It adds a `TestScheduler_Outbound` row where a fake socket answers with a foreign `HubPath` holding a quote and a line break, and with a set `Code`, then asserts the notice text. That needs `listenHubFake`'s path to be settable by the row.

F3. LOW, consistency
- Location: card 1 refusal text and corrupt-record way forward (`_lym/plan/01-hub-registry.md:51,56`), against card 5 (`_lym/plan/05-hubs-verb.md:25`) and card 4 (`_lym/plan/04-hub-directory.md:72,86`).
- Summary: "run any lym creel verb … in that hub once" does not always re-record the hub, so the way forward fails on two reachable paths.
- Evidence:
  - `lym creel hubs` is a `lym creel` verb, and card 5 says it never starts a daemon and never writes the record.
  - Against a live daemon that recorded less than `hubRecordInterval` ago, a verb records nothing, and card 4's row pins that. So removing a corrupt file and running one verb within the minute leaves the hub unrecorded.
- Fix: keep D1's `lym creel inbox` example as the named step, and have card 1's text or card 4's interval say a verb past the minute. Alternatively, card 4 can reset the interval when a record finds the entry missing.

Tool calls:
1. One Bash call read `state.UpdateJSON` and `ReadJSON` (directory creation and decode errors), the rest of `daemon.dispatch` (whether each verb reaches the guarded `Store` method), every `.From` and `eventReturned` site (recurrence of the step back), `Store.Withdraw`, and the `LineFits` and `MessageLine` callers. The manifest's reads stopped short of each of these.
