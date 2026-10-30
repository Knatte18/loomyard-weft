```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Per behaviour, from test names and rows in the diff. I did not read the test bodies.**

| Behaviour | Pinning test |
|---|---|
| D3: ring claims for a marked waiter, with budget, hold, field cap and journaling at the ack | `AMarkedWaiterIsRungWithTheBudgetedClaim…`, `…UnderAnotherSendersHold…`, `…WithoutALineOverTheHookFieldCap` (`deliver_test.go`) |
| D3: ring starts the Stop count over | `AMarkedRingStartsTheStopCountOver` |
| D3: pause, failed pause read, or a claim that took the set first rings nothing and keeps the waiter | `APauseReadAtTheRing…`, `APauseReadErrorAtTheRing…`, `AClaimTakingTheSetBeforeTheRing…` |
| D3: ring outcome never taken is released on `Cancel` | `ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled` |
| D3: released ring claim leaves the mail unread and the bell follows | `AReleasedRingClaimLeavesTheMailUnreadAndTheBellFollows` |
| D3: ring ack journal failure leaves the ring unanswered | `AJournalFailureAtTheRingsAckLeavesTheRingUnanswered` |
| D3: marked hook-wait rung with mail and acked, over the socket | `AMarkedHookWaitIsRungWithTheMailAndItsAckJournalsTheRead` (`server_test.go`) |
| D3: client leaves before the ack, mail stays unread | `AMarkedHookWaitWhoseClientLeavesBeforeTheAck…` |
| D3: no resume once the ring reached the handler | `AHookWaitWhoseConfirmationIsLostAfterItsRingReachedTheHandlerIsNotResumed` |
| D3: `ErrAckUnsent` when the ack write fails | `AHookWaitWhoseAckCannotBeSentReturnsErrAckUnsent` |
| D3: ack journal failure releases the unjournaled ids; a read's ack keeps them reserved | `AJournalFailureAtTheAckReleasesEveryIdItLeftUnjournaled`, `AJournalFailureAtAReadsAckKeepsTheIds…` |
| D3 `hookWaitExit` rows | `TestHookWaitExit` (6 rows) |
| D3 hook-wait: exit 2 with lines on stderr; subagent or refused call exits 0 | `cli_test.go` subtest "hook-wait rings a strand as exit 2…" |
| D3 hook-wait: stderr write fails, exit 0, mail unread | `cli_test.go` subtest "hook-wait whose stderr refuses the lines…" |
| D3: unmarked waiter rung with no mail (older client) | **none found** in the names I saw. The "AMarkedHookWait…" rows are all marked. |
| D3: ring with a hold, pause, or empty claim | The empty-claim branch is unreachable (see the handler's reading of `wakeHookStrand`), so no test is owed. |
| D3 test seam: ring answered but the client leaves or the daemon stops before the outcome is written | Only the Cancel-drain test above. **No test of `awaitOutcome`'s select with an outcome already in the channel** (`server.go` ring branch), though the record's "Notes for the plan writer" asks for one. |
| D3: write failure of the ring response in `awaitOutcome` (the `_ =` write ignored) leads to release | **none found** |
| D6: echo left at the ring ack and handed out ahead of mail | `AnAcknowledgedRingLeavesAnEchoTheNextClaim…` |
| D6: pause or hold does not keep the echo; Stop cap does not keep the echo | `ThePausedOrHeldStrandsClaimStillHandsOutTheEcho`, `TheStopCapKeepsTheMailBackButNotTheEcho` |
| D6: shared budget, first item always handed out, release keeps the echo | `TheEchoSharesTheBudgetAndAReleaseKeepsIt` |
| D6: ack failure on a ring echoes only the ids before the failing write | `AJournalFailureAtTheRingsAckEchoesOnlyTheIdsBeforeTheFailingWrite` |
| D6: ack failure on a claim drops the echo it handed out | `AJournalFailureAtAClaimsAckDropsTheEchoItHandedOut` |
| D6: skipped id still echoed | `ARingAckEchoesAnIdThatLeftTheHolderBeforeTheAck` |
| D6: owed echo keeps the flag, including while reserved | `AnOwedEchoKeepsTheFlagUntilTheClaimThatShowsItIsAcknowledged` (`flags_test.go`) |
| D6 echo-only claim neither marks the ring answered nor counts a Stop | **none found by name.** The echo tests may assert it, but no name says so. |
| D6 `hook-read` echo rendering and exit | `cli_test.go` subtest "after a ring hook-read shows the echo…", `hookread_test.go` `FlagStandsAndAnEchoAloneIsWritten` and the `hookReadExit` rows |
| D1/D2/D6 JSON shapes | `claudeengine` `HookOutput` rows: ToolBoundary and Stop, each with mail, mail+echo, echo alone |
| D1 "Stop with nothing exits 1, prints nothing" | The `cli_test.go` "prints nothing when none is left" subtest |
| D4 rewake keys on the waiter entry only | `settings_test.go` was changed (+39 lines). I saw no names, so I take it as covered. |
| Merged Stop command records the turn end on exit 1 with echo output kept | `hookcmd_integration_test.go` changed (+15). I did not verify the echo case. **Unverified.** |
| D5: log line per claim path with the right outcomes and levels | `TestStore_ClaimLog` rows: tool, stop, released, empty, ack-failed (claim and ring), echo, ring acknowledged and released, paused ring, and the nothing reasons paused, held, stop-cap, stop-count-unknown, nothing-claimable |
| D5: bell log | The bell rows `ABellSentAfterTheGrace…`, `ABellWhoseSendDidNotLand`, `ABusyBell`, `AFailedBellNamesItsError` |
| D5: ack-failed is a Warn and a claim handing out something is Info | **Unverified.** I only saw names, and `TestStore_ClaimLog` may assert levels. |
| `takes_mail` tolerance by an older daemon | **none**, and none is needed. It is a plain unknown-key ignore. |
| Smoke tier3 | `smoke_llm_test.go` changed (+146). I did not check its subtests against D1 to D3. |

**Findings**

1. MEDIUM, class design. The ring answer-path ends are not tested through the real `awaitOutcome`.
   - Behaviour: "client leaves or daemon stops after the ring was answered but before the outcome was written releases the claim." This is D3 acceptance and a focus item.
   - Location: `internal/creelengine/server.go` `awaitOutcome` ring branch, `d.done` and `watcher.closed` cases.
   - Evidence: the only related rows are `ARingOutcomeNeverTakenIsReleasedWhenItsWaitIsCancelled` (a store-level `Cancel`) and `AMarkedHookWaitWhoseClientLeavesBeforeTheAck…` (client leaves before the ack, not before the outcome write).
   - The fasit's note asks for a test seam that orders an already-queued outcome before the close or the shutdown. I saw none in the test names or `export_test.go` names.

2. LOW, class design. No test named for an unmarked waiter rung without mail.
   - Behaviour: an older client's `hook-wait` has no `takes_mail`, so the ring carries no mail and claims nothing. This is D3 acceptance and a focus item.
   - Evidence: no row for it in the `deliver_test.go` or `server_test.go` names I read.

3. LOW, class consistency. No test named for the D6 rule that an echo-only claim neither answers the ring nor counts a Stop.
   - Evidence: no row by name in `deliver_test.go`.
   - The doc states the rule at `creelengine/doc.go`.
   - It may sit inside `AnAcknowledgedRingLeavesAnEchoTheNextClaim…`, but I did not read the bodies.

**Not verified because I made no body reads:** the level assertions in `TestStore_ClaimLog`, the D4 rows in `settings_test.go`, the echo case in `hookcmd_integration_test.go`, and the `smoke_llm_test.go` subtests.

**Test economy:** from names alone I saw no duplicate coverage.
- `AJournalFailureAtTheAckReleasesEveryIdItLeftUnjournaled` (`server_test.go`) and the claim ack-failed log rows both target the claim's failing-ack release. One pins the daemon's release behaviour and the other the log line, so I do not treat them as duplicates.

Tool calls: 1. `git diff` of the test files, filtered to test and row names, because my context held only the production diff and the function-name grep.
