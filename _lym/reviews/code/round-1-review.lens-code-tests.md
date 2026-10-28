```yaml
lens: code-tests
outcome: reported
```

Task: list per behaviour the test that pins it, or none, for the code-tests lens.

**Per behaviour → pinning test**

- `orchroute.Asker`/`From`/`FromPair` (D1, D3 beam and pair) → `TestAsker` (`internal/orchroute/asker_integration_test.go`), `TestLiveFrom`, and the added beam row in `internal/orchcli/notify_integration_test.go`.
- Three notifier wirings use `NotifyAsker` (D3) → **none** (see F1).
- `post --parent` resolves, refuses when no orch is recorded, and refuses with a receiver or `--reply` → the `creelcli` `cli_test.go` rows "post --parent posts to the parent orch…" and "post refuses a wrong form before any call…".
- `--expect-reply`, the mark on `read`, the hook line and the log entry; `--wait-reply` implies the mark; `--reply` refuses the flag → `PostWith_MarksTheMessage…`, `ClaimedLineOfAMarkedMessage…`, `Reply_LogEntryCarriesTheIdItAnswers`, the `server_test.go` row, the `readtext_test.go` row, and the `cli_test.go` "post --expect-reply marks…" row.
- A reply clears the mark → `ARepliesStopTheStepsAfterItAndALateReplyIsStillDelivered`.
- Chase clock (running only, pause stops it, dormant holder, another sender's hold stops it, own sender's hold does not, `chase-aged` persistence) → `TestScheduler_Chases` rows `AChaseAgedRecordFollowsEachRunningTick…` and `ADormantHolderAndAnotherSendersHold…`.
- Restart (no third chase, overdue steps one interval apart, no paused time) → `AReopenAfterAPause…`, `ASeededChasePostWithNoChasedRecord…`, `ASeededClockPastTheNextStep…`.
- Two chases then `unanswered`, with the sender notice and the waiter answered → `ProgramSendersMessageIsChasedTwice…`, `StrandSendersMessageReadThroughReadAndAck…`, and the `cli_test.go` `--wait-reply` exit-1 row.
- No notice to a never-read program, to a removed strand or to EXTERNAL → `NoNoticeGoesToAProgramThatNeverRead…` and `AnExternalSendersMessageIsChased…`.
- A never-read message and a program holder are not chased → `AnUnreadMarkedMessageAndOneAProgramHolds…`.
- Chase content-free, naming EXTERNAL by its bare name → the EXTERNAL row.
- Reply to a chase answers the chased message → `AReplyToAChaseAnswersTheChasedMessage…`.
- Forward of a chase refused → `AChaseTheHolderStillHoldsUnread…`.
- Reply to a creel notice refused → `ASenderNoticeOfAnUnansweredOutcomeTakesNoReply`.
- Sweep widening and `holder-gone`, including a holder with no other mail → `HolderRemovedAfterReadingAMarkedMessageSettlesItUnansweredHolderGoneWithNoChase`.
- Retention carries a chaseable message → the `journal_test.go` row "a chaseable message, an unread chase and a program's read are carried…".
- `lym report replies` → `TestComputeReplies` and `TestRepliesVerb`.
- Hold notice text → `hold_test.go` and `attach_asking_test.go`.
- `ErrNoRunningTurn` and the `Runner.Inject` refusals name creel verbs → `run_test.go` (`wantMsg: "lym creel post"`) and `run_inject_test.go`.
- Removed shuttle verbs → `NoChild_*` in `shuttlecli/cli_test.go` and `removedverbs_test.go`.
- `RequirementPeerMessaging` removed → `requires_test.go`.
- `SendMessage` scan → `TestAgentMessaging_NoEngineTool`.
- The seat/chair/advisor prompt tests were updated.
- Trust sentence in `claudeengine/settings.go` (D6) → **none** (see F2).
- Chase refusal for a former holder → no reaching test, by decision (D10 amendment).

**Findings**

- **F1 — MEDIUM, design** — `internal/burlercli/wiring.go:152`, `internal/loomcli/wiring.go:464`, `internal/webstercli/wiring.go:211`: all three notifier wirings changed to `orchcli.NotifyAsker`, and no test in those packages reaches `SetNotifier` or `NotifyAsker`. Reverting any one of them to `NotifyParent` fails no test. Only the `orchroute`/`orchcli` walk rows cover the rule. The D1/D3 acceptance criterion "hold notice and `--parent` resolve the same strand" is therefore not pinned at the wiring. Evidence: the grep for `orchroute.|SetNotifier` in the three packages' `_test.go` files returned nothing.
- **F2 — LOW, design** — `internal/shuttleengine/claudeengine/settings.go` (D6 trust sentence reworded): the diff touches no test in `claudeengine`, only `capabilities_test.go`. The sentence is static prose, so this is exempt unless an existing test asserts it. Reported only because I did not confirm that one exists.
- **F3 — LOW, design** — `internal/creelengine/journal_test.go`: one combined row covers retention. I saw no assertion that `MarkSeconds` and `ChaseConsumed` survive a carry and then drive the next step after a reopen. The seeded-clock rows use journal records, not carried ones. If that is missing, breaking the carried-snapshot clock fields fails no test. Evidence: the diff stat of the carried row does not show it.

**Tool calls**
1. `git diff -U0` over `*_test.go`, to list the new test names and rows.
2. One combined read: the `hold_test.go` diff and the three wiring diffs, a grep for `orchroute.|SetNotifier` in the three packages' tests (no hits), a grep of `replies_test.go` for the counting rows, and a grep of the `shuttleengine` tests for the creel-verb refusal messages. The context did not hold the test bodies.
