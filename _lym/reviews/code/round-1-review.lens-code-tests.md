```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that would fail if it broke, or none. I read test names and row titles only, not test bodies.

**Behaviour → pinning test**

Creel
- Take returns pending machine mail oldest first with its reply-to: `TestStore_MachineOutbound` / `TakeAnswersTwoPendingMessagesOldestFirstWithTheirReplyTo`.
- A take after the hour returns the message and never hands it out: `ATakeAfterTheHourReturnsTheMessageAndNeverAnswersIt`.
- Report journals only on a changed reason: `AnUnchangedHoldReasonReportedThreeTimesWritesOneRecordAndAChangedOneASecond`.
- Delivered messages are not returned at the hour, and a returned-settle for them writes nothing: `ADeliveredMessageIsNotReturnedAtItsHour…`.
- The hour timer returns messages with the relay down, using the last hold reason or `bridge-unsettled`: `WithTheSchedulerStartedAndNoTake…`.
- Two due messages go in one pass, and the loop never waits under a second: `TwoMessagesDueTogether…`.
- A late settle journals delivered and posts one notice: `ASettleDeliveredAfterTheReturn…`.
- A settle returned with a Slack code, such as `channel_not_found`: `ASettleReturnedWithChannelNotFound…`.
- Take, report and settle refuse every sender but the relay: `TakeReportAndSettleAreRefusedToEveryoneButTheBridgeProgram`.
- Deliver refuses every sender but the relay: `ADeliverFromAnOrchStrandAnotherProgramOrExternalIsRefused`.
- An unknown id is refused: `ReportAndSettleNamingAnUnknownId…`.
- Deliver stores trusted or untrusted mail and dedupes: `ADeliverFromAnOrchStores…`, `DeliversOfTheSameSendingId…`, `ADeliverFromAnotherRole…`.
- Reply-to checks: `AReplyToANeverSentId…`.
- Deliver refusal rows: `ABadSenderOrReceiverOrBody…`.
- A reply journals machine-outbound and starts no per-hub worker: `AReplyToMachineMail…`.
- Reply refusals: `ARefusedSendReady…`.
- Forward guard: `AForwardToAProgramOrToAnotherHub…`.
- The sweep journals machine mail undelivered: deliver_test `AGoneHoldersUnreadMachineMail…`.
- Post refusal rows, including own-name-before-check, program, external, non-orch sender and receiver, send-ready, line cap and empty body: `verbs_test` rows.
- inject, interrupt, hold, release and post --interrupt refuse a machine name: the verbs_test row.
- Forward to a machine name is refused: `Forward_ToAnotherMachine…`.
- The machine stamp in a claimed line: `ClaimedLineOfMachineMail…`.
- Journal fold, delivered-after-returned and the inbound fold: four `journal_test` rows.
- A creel-side CLI round trip: `cli_test` subtests.
- `agentname` parser: `TestParseMachineAddress`, covering own name before the checker, former name, bad machine part and empty parts.
- Bridge seam `SendReady`: `TestSlackBridge`.

Relay
- Send-side holds, send refusals and the poll: `TestBridgeSend` subtests.
- Records tolerate unknown keys: `RecordsKeepAKeyTheyDoNotKnow…`.
- Receive path: `TestBridgeReceive` subtests.
- Peer commands: the `inbound_test` subtests.
- Holder routing and the per-interval resync hint: the `holder_test` subtests.
- Operator re-invite, plus archive on retirement: the `keeper_test` subtests.
- Envelope and channel name: `TestBridgeMessage` and `channelname_test`.
- API calls: the `api_test` subtests.

**Findings**

1. **MEDIUM, design (coverage gap)**, `internal/creelengine/journal_test.go` and the `machinemail_test.go` rows.
   - Summary: nothing pins that a pending machine-outbound message survives the retention carry with its receiving machine and last hold reason (D3, D8). Carrying means the `carryRecords` / `applyCarried` `of: machine-outbound` path, with the hold reason in `Reason`.
   - Missing: a carried machine-inbound message with `of: machine-inbound`.
   - Missing: an older fold that reads either event as an id only.
   - What I saw: the `journal_test` row titles cover machine-outbound, machine-held, machine-delivered and machine-inbound folds, but none names carry or retention.
   - Evidence: `fold.go` carry hunks at `carryRecords` and the `applyCarried` `Of: eventMachineOutbound` case.
   - Caveat: I did not read the bodies. If a carry-day test exists inside `TestJournal_Retention`, this finding is void.

2. **MEDIUM, design (coverage gap)**, `internal/creelengine/machinemail_test.go`.
   - Summary: no row names the "no per-hub outbound worker starts for a machine-outbound message at the scheduler's start or on a post" criterion for a post.
   - What is covered: the reply row says it starts no hub. The scheduler start-scan case is not visible.
   - What a break would do: calling `wakeOutboundLocked` from `postMachine` would not fail any named row.
   - Evidence: `crossmachine.go` `postMachine`.

3. **LOW, design (coverage gap)**, `internal/creelengine/verbs_test.go`.
   - Summary: nothing pins the `Reply` path for a message whose `SendingMachine` is set and whose replier is a non-orch. The `ARefusedSendReadyAndANonOrchReplier…` row covers it, so this is probably fine.
   - Nothing pins `WaitReply` for a machine-outbound id (`verbs.go` `WaitReply`) or the `settledOutcomeLocked` machine branch in `pool.go`. The `--wait-reply` return and reply outcomes (OutcomeReturned, OutcomeReply) for machine mail appear in no row title.
   - Evidence: `pool.go` `settledOutcomeLocked` machine branch; `verbs.go` `WaitReply`.

4. **LOW, design (coverage gap)**, no test seen for `machineHoldWayForward` wording per reason, which is the return notice's content. The late-notice row and the CLI test may assert it.
   - Evidence: `machinemail.go:229`.

5. **LOW, design**: the hubgeom test and the slackcli wiring.
   - `hubgeom.TestSlackMachine` is contained. Per the report it is covered by `TestSlackGeometry`.
   - Its only new assertion is the `PeersPath` line.
   - `TestSlackGeometry` carries the same `PeersPath` assertion in the same diff.
   - Finding, test economy: fold the `TestSlackMachine` row into `TestSlackGeometry`.
   - Coverage blocks do not see assertions, so check the contained test's other assertions first. If it pins `SlackMachine`'s other paths that `TestSlackGeometry` does not, accept it and mark it `//testtiming:keep`.
   - Evidence: `internal/hubgeom/hubgeom_test.go` hunks at lines 533 and 583–588.
   - I found no sign that `TestSlackMachine` is pinned to something `TestSlackGeometry` misses.
   - Not covered: `internal/slackcli/wiring.go` `programMailbox` machine passthroughs, and `internal/creelcli/goclient.go`. The `cli_test` subtests exercise the goclient, and the wiring has no test of its own. Neither is a behavior beyond forwarding, so I raise no finding.

6. **LOW, design (static prose)**, `cmd/lym/daemonwakeups_test.go`: it gains one list entry (`machinemail.go` and `bridgepoll.go`). That is a source list for the wakeup scan, not an assertion on prose.

**Tool calls (4)**
1. Test-name and row grep over the creelengine, creelcli and agentname test diffs, for coverage by title. My context held no test files.
2. The same grep over the slackengine test diffs.
3. A title grep of `machinemail_test.go`, plus the `verbs_test` and `deliver_test` diffs, for the refuse-verb, scheduler and hook coverage. My context held no test files.
4. A grep of `machinemail_test.go` rows and scheduler uses, to confirm the hour-timer and start-scan coverage.
