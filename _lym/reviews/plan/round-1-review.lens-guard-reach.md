```yaml
lens: guard-reach
outcome: reported
```

Guard-reach lens: whether each guard, refusal, branch and test row the cards add can be reached from a real entry point, and whether a guarded site recurs unguarded.

**Per guard, reachable or not**

- Card 1, `ParseMachineAddress` steps 1–4 (split, own-name before `checkMachine`, machine rule, name part): reachable. They are reached through the public parser, and through `post` because the CLI passes the receiver raw (`internal/creelcli/cli.go:441`). `ly::orch` reaches step 4.
- Card 3, `Store.post` routing of an `IsMachineAddress` receiver with `--interrupt` through `refuseOtherHub`: reachable.
- Card 3, `refuseOtherHub` for `::` names: reachable. `inject`, `interrupt`, `hold` and `release` all call it first (`verbs.go:1091`, `1211`, `1328`, `1453`).
- Card 3, `Forward` refusing a `::` receiver: reachable once the row holds an unread message.
- Card 3, `postMachine` steps:
  - step 1, nil bridge: a fail-loud wiring guard, reached by a row anyway.
  - step 2, `Names` error: reachable through `FailNames`.
  - steps 3–7 (parse, non-orch sender, non-orch receiver, `SendReady`, line cap): reachable.
- Card 3, `WaitReply` on a machine-outbound id from another strand: reachable.
- Card 4, `requireBridgeProgram` (orch, other program, scoped `SLACK`, EXTERNAL): reachable. Its "of this hub" clause is refused upstream by `checkProgramClaim` (`identity.go:114`), but no row names it, so no finding.
- Card 4:
  - Unknown-id refusals in `ReportMachineHold` and `SettleMachineDelivered`: reachable.
  - A report on a settled message writing nothing: reachable.
  - A late settle delivered after the return: reachable.
  - `TakeMachineOutbound`'s due-first return: reachable.
  - The hour loop's floor of one second: reachable.
- Card 5, `DeliverMachine`:
  - step 1, program check: reachable.
  - step 2, sender names this machine: reachable through the verb by any `SLACK` claim, though the relay skips such mail first.
  - step 3, receiver no orch of the hub: reachable at the verb, but not from the relay (finding 2).
  - steps 4–7 (line cap, dedupe, reply-to): reachable.
- Card 5, `Reply` sender checks:
  - non-orch replier: reachable after a forward.
  - `SendReady` refusal: reachable.
- Card 5, `Forward` of machine-inbound mail to a program or another hub's strand: reachable.
- Card 5, `releaseGoneHolder` widening to `SendingMachine`: the state is already caught by the existing `!isStrandName(next)` branch (finding 4).
- Card 6, `SendReady` step 1, the pair's orch: reachable. `DobbyStart.Beams` lists only pairs carrying the beam mark (`internal/hubgeom/dobbygeom.go:95`), so a task pair's `&lt;s&gt;:&lt;slug&gt;:orch` passes `isOrchName` and reaches it.
- Card 6, `SendReady` steps 2–3 (button off, `RequireConfigured`): reachable.
- Card 9, `parsePeerCommand` in an orch channel: reachable.
  - Own-name refusal: reachable.
  - `CheckName` refusal (`Bad_Name` lower-cases to `bad_name`, which keeps the `_`): reachable.
  - None or several candidates, unknown app id, revoke with no pin: reachable.
  - `users.list` refusal through `retryOrGiveUp`: reachable.
  - A non-operator is skipped by `handle`'s first check: reachable.
- Card 10, `sendOne`:
  - `button-off`, while another button keeps the relay up: reachable.
  - `peer-unapproved`: reachable.
  - `ensureBridge` holds (`bridge-archived`, `bridge-name-taken`, adopt by either bot): reachable.
  - Refused peer invite: reachable.
  - Post refusal count, `refusalCap`, drop on `is_archived` or `channel_not_found`: reachable.
- Card 10:
  - `settleAccepted` drop on refusal: reachable.
  - A taken id already in `Accepted`: reachable.
  - A report refused for a returned id: reachable.
- Card 11, holder route of a bridge-map channel and the per-hub, per-base-interval resync hint: reachable.
- Card 11, `takeHints` bridge, thread and resync signals: reachable.
- Card 11, `handle` steps 1–13: reachable.
- Card 11, the operator branch in a bridge channel running `runPeerCommand`: reachable, but its failure path is unguarded (finding 1).
- Card 11, resync branches (approved creator with live or retired orch, unpinned creator's notice in the prime channel or each open channel): reachable.
- Card 12, bridge invite refusal, bridge retire, refused archive retried later: reachable.

**Findings**

1. **MEDIUM, design.** `_lym/plan/11-bridge-receiving.md`, step 2 (`bridgeReader.operator`).
   - Summary: card 9 guards `runPeerCommand`'s failures (a Slack refusal, an unwritten peers record) with `retryOrGiveUp`. Card 11 calls the same `runPeerCommand` for a bridge-channel `approve` or `revoke` and states no failure path, so on a failure it is unsaid whether the operator cursor moves, losing the command, or holds with no give-up.
   - Card 11 also calls `in.runPeerCommand` and its `answer`, `episodes` and `failingKey` from the bridge-poll goroutine. `relay.go:452` says one goroutine, the wake loop, owns the inbound's reads and episodes, so the guarded state now has a second, unguarded writer.
   - Evidence: `_lym/plan/09-approve-and-revoke.md:46`, `_lym/plan/11-bridge-receiving.md:80-81`, `internal/slackengine/inbound.go:702-715`, `internal/slackengine/relay.go:451-452`.
2. **LOW, consistency.** `_lym/plan/11-bridge-receiving.md`, row "a deliver creel refuses (a receiver that is no orch of the hub) is skipped with one notice".
   - Summary: in the relay, the receiver is always the channel's local orch after step 10, and a live orch of this hub after step 11. Creel's `isHubOrchName` refusal (card 5, step 3) cannot be reached from the relay, so the row's named cause is a scripted state that cannot occur.
   - The reachable relay-side refusal is the reply-to mismatch (card 5, step 6), which the row should name.
   - Evidence: `_lym/plan/11-bridge-receiving.md:90-94,116`, `_lym/plan/05-machine-inbound-deliver.md:52`, `internal/creelengine/pool.go:242-245`.
3. **LOW, design.** `_lym/plan/05-machine-inbound-deliver.md`, step 6 (reply-to check).
   - Summary: the check matches the machine-outbound entry's `To` "as written" (card 3) against the envelope's `from`. An orch that addressed the peer by one of the peer's former names, which D8 has the peer accept, gets its reply from the peer's current name. Creel refuses that legitimate reply, and the relay skips it with a notice.
   - Evidence: `_lym/plan/03-machine-outbound-post.md:72`, `_lym/plan/05-machine-inbound-deliver.md:55`, `_lym/discussion/decision-record.md:351`.
4. **NIT, consistency.** `_lym/plan/05-machine-inbound-deliver.md`, `releaseGoneHolder`.
   - Summary: unread machine-inbound mail with no forwarder already takes the undelivered branch, because `next = msg.From` (`osl::zz:orch`) fails `isStrandName`. The widened `SendingMachine` condition guards a state the existing branch already reaches. The row passes with or without the edit.
   - Evidence: `internal/creelengine/sweep.go:175-178,239`.

**Tool calls**
1. `PostWith`, the CLI post receiver handling, the callers of `refuseOtherHub` and the `daemonRefusal` wrapping: to confirm no upstream check rejects a `::` receiver, and how a refusal reaches the relay.
2. The inject and interrupt verb heads, the CLI post body, `waitsForHubOrch` and `isStrandName`: to confirm guard order and the sweep's existing branch.
3. `internal/hubgeom/dobbygeom.go` `DobbyStart.Beams`: to confirm a task pair's orch is not listed as a beam, so `SendReady` step 1 is reachable.
