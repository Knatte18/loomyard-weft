```yaml
lens: way-forward
outcome: reported
```

Review of the plan's refusal, hold and recovery steps, checking that each named next step can succeed (way-forward lens).

**BLOCKING**
- **Card 10, D7 at-least-once posting: a message can be posted twice within one relay run.**
  - Evidence: card 10 `sendOne` step 5 writes `Accepted` and then settles, keeping the entry while the settle fails on a lost connection. Only `settleAccepted` reads `Accepted`, and `RunRelay` calls it once, before the poll.
  - The next poll's `send` runs `TakeMachine` afresh. Creel still holds the message pending, so it comes back. `sendOne` steps 1–5 never check `Accepted`, so the message is posted again.
  - This breaks D7's "a restart never posts the message again once the record is written". The double post also happens within a single run.
  - Fix: before posting, `sendOne` settles delivered any taken id already in `Accepted` instead of posting it. Alternatively, `send` runs the `settleAccepted` pass first on every round.

**MEDIUM**
- **Card 3, D3/D5: the over-cap refusal names a way forward that cannot work across machines.**
  - Evidence: `postMachine` step 7 refuses through `lineOverBudget` (`internal/creelengine/verbs.go:61-63`), which says "write it to a file and mail the file's path".
  - A path on this machine is unreadable for the peer orch on the other machine.
  - Fix: give the machine refusal its own wording, such as "shorten the message or split it into several posts".
- **Card 5, D2/D3: `Reply` to machine-inbound mail skips the orch-sender check and the button/token/setup check.**
  - Evidence: card 5's `Reply` paragraph runs only "the same line check" before journaling `machine-outbound`. Card 3's checks 4 and 6 (orch sender, `bridge.SendReady`) do not run on this path.
  - D2 says `post --reply` carries the cross-machine post rules. D3 says a post is refused at once, before journaling, when the button is off or a token or the setup is missing.
  - A reply with the button off journals and then comes back an hour later instead of being refused with "press the button".
  - A machine-inbound message forwarded to a local non-orch strand, which `Forward` allows, lets that strand reply across machines.
  - No row covers either case. Fix: the reply path runs the orch check and `SendReady`, with rows for both.
- **Card 4, D3: the hour's return notice names a way forward that cannot clear a held message.**
  - Evidence: card 4 re-signs `noticeReturnedLocked` so the machine return shares the per-hub wording (`internal/creelengine/outbound.go:169`): "lym creel log --id … then post it again or decide without the answer".
  - For a message returned with `peer-unapproved`, `bridge-name-taken`, `bridge-archived` or an invite code, posting again only holds it again until the operator acts.
  - Fix: the machine notice names the hold reason's own way forward, such as `approve <machine>` in an orch channel, or the relay notice to look for.
- **Card 10, D4: a hold episode's notice mark is never cleared when creel returns the held message at its hour.**
  - Evidence: card 10 says "Any hold ending, by a post or a changed reason, clears its key."
  - A message held for `peer-unapproved`, `bridge-name-taken` or `bridge-archived` that creel returns at the hour simply drops out of `TakeMachine`. Nothing clears its mark in the persisted `bridgeNotices`.
  - The next message of the same pair and kind, perhaps days later, then gets no notice, so the operator never learns of it.
  - Fix: end the episode when no taken message of the pair is held for that kind, and add a test row for it.
- **Card 11, D8: deliver refusals other than reply-to have no stated handling.**
  - Evidence: card 11 step 13 covers only "A refusal (`IsDaemonRefusal`) of a reply-to is skipped with a notice" and "a lost connection holds the cursor".
  - Card 5 `DeliverMachine` also refuses a sender naming this machine, a receiver that is no hub orch, a line over the cap, and a sender that is not `SLACK`.
  - If such a refusal holds the cursor, the channel stays stuck for good. A held message sits outside `retryOrGiveUp`'s five minutes, and these refusals never clear.
  - Fix: every answered refusal is skipped with a read-side notice naming the reason, and a row covers it.
- **Card 10, D4: the `bridge-name-taken` hold's notice names no way forward.**
  - Evidence: card 10 `ensureBridge`, "Any other creator holds `bridge-name-taken`, with one notice naming the channel". It gives no next step.
  - The keeper's notice for the matching orch channel names one (`internal/slackengine/keeper.go:204`): "rename that channel in Slack, which frees the name".
  - The hold clears only once the name is freed, so the notice should say so. The next create, at most once per ceiling, then succeeds.

**LOW**
- **Card 10: notice destinations and the operator invite are unspecified.**
  - Card 10 step 2 names where the `peer-unapproved` notice goes (O's channel and mail to O). The `ensureBridge` hold notices (`bridge-name-taken`, `bridge-archived`) and the refused-peer-invite notice have no stated destination.
  - A refused operator invite into a new bridge channel has no stated notice or retry. Card 12 only covers a change of operator.
- **Card 11 step 8: the wrong-app notice names the app but no next step.**
  - The way forward would be `approve <machine> <app id>` to re-pin, when the operator finds the app is the real peer.
- **Card 9: two gaps in `approve`.**
  - `approve <m> <app id>` with no bot of that app id "refuses, naming the id" and gives no next step. It should point at the other machine's `status`.
  - Card 9 relies on `BotUsersNamed` with an empty name keeping every bot. Card 8 defines `BotUsersNamed` as keeping the bots "whose `name` equals name" and states no empty-name case. The step card 9 depends on is undefined on the card that creates the method.
- **Card 3: two refusals with a weak or missing next step.**
  - The nil-bridge wiring guard's way forward ("post from a hub daemon started by lym") cannot succeed before card 6 wires `Options.Bridge`.
  - A name part that does not parse is refused with `agentname.Parse`'s grammar error and no way forward (card 1 step 4).
- **Card 4: no relay step after a refused `ReportMachineHold`.**
  - `ReportMachineHold` refuses an unknown id. Card 10 does not say what the relay does on that refusal: whether it drops its `Refusals` entry or retries.

Tool calls: none beyond my context.
