```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

Inbound (`inbound.go`)
- Operator top-level message forwarded as `[slack]`, markup unescaped, links turned into URLs: `TestInboundPoll` rows `OperatorMessageReachesThePrimeOrch` and `SlackMarkupReachesTheOrchAsPlainText`.
- Non-operator, bot, subtype and thread-reply messages move the cursor only: `WhatIsNotTheOperatorsTopLevelTextMovesTheCursorOnly`.
- `@slug` routing and unknown-slug answer: `AtSlugReachesThatBeamsOrch`, `UnknownBeamIsAnsweredWithTheBeamsAndPostsNothing`.
- A `Hub.Beams` error leaves the cursor before the message and the message is forwarded once on retry: `BeamsErrorRetriesAndForwardsOnce`.
- Missing cursor, empty channel and changed channel forward no history: `FirstPollWithNoCursorForwardsNoHistory`, `EmptyChannelFirstPollStartsAtTheClock`, `ChangedChannelForwardsNoHistoryOfThatChannel`.
- A refused post holds later messages, with the 5-minute give-up answer: `RefusedPostHoldsLaterMessagesUntilAcceptedOrFiveMinutes`, subtests `AcceptedWithinFiveMinutes` and `PastFiveMinutesIsAnsweredOnceAndPassed`.
- 4 KiB cap at and over the limit: `BodyOverCreelsCapIsAnsweredNotPosted`.
- Paused receiver gets the post plus one answer: `PausedReceiverGetsThePostAndOneAnswer`.
- A `Hub.Paused` error after an accepted post moves the cursor and sends no answer: `PausedErrorAfterAnAcceptedPostStillMovesTheCursor`.
- `away`, `back` and `status` commands, failure retries and escaping: `PresenceCommandsSetAndReadThePresenceAndAreAnswered`, `AwayWhoseBeamsReadFailsAfterTheWriteIsAnsweredAndPassed`, `AwayWhoseWriteFailsIsRetriedUntilTheWriteWorks`, `StatusOverAnUnknownPresenceIsRetriedThenAnsweredWithTheError`, `AnswerIsEscapedForSlack`, `AnswerSlackRefusesStillMovesTheCursor`.
- Presence changed by another writer counts as activity, and one notice per Slack refusal episode: `PresenceChangedByAnotherWriterIsActivity`, `SlackRefusalMailsOneNoticePerEpisode`.
- `SetPresence` notices: `TestSetPresence`, four rows.
- History paging to `has_more` false, oldest first: `TestAPI` History case, `api_test.go:130-149`.

Outbound (`outbound.go`)
- Sender filter, escaping and beam answer hint: `MailFromOrchProgramAndReplyIsPostedOnceAndAcknowledged`, `TextIsEscapedAndBeamMailCarriesTheAnswerHint`, `NonOrchStrandAndExternalMailIsDroppedAndAcknowledged`.
- Slack failure mid-read posts each message once: `SlackFailureMidReadPostsEachMessageExactlyOnce`.
- The 5-refusal replacement line: `MessageSlackRefusesFiveTimesIsReplacedByOneLine`.
- Transport and 5xx failures never count toward the cap: `TransportFailureAndServerErrorNeverCapOutAMessage`.
- Posting budget and release: `BacklogPastThePostingBudgetIsReleasedAndPostedOnceAcrossReads`.
- 24-hour skip with one summary line: `MailOlderThanADayIsAcknowledgedUnpostedWithOneCountLine`.
- Refused acknowledgement keeps the record: `RefusedAcknowledgementKeepsTheRecordSoNothingIsRepostedTwice`.
- Rate limit, empty wait and timeout after the handler ran: `RateLimitReturnsItsRetryAfterAndRaisesTheFloor`, `EmptyWaitBacksOffAndKeepsTheRecord`, `TimeoutAfterTheHandlerRanKeepsTheRecordAndReturnsTheFloor`.

Relay and start
- Lock, first read, strand-pane refusal ending the relay, other refusals retrying, re-exec and `LastSlackOK`: `TestRunRelay`, rows at `relay_test.go:146-460`.
- Inbound wait doubling and floor raise: `InboundWaitDoublesToTheCeilingResetsOnActivityAndARateLimitRaisesTheFloor`.
- Restart cap by lifetime, including the 10-minute boundary: `TestEnsureRelay`, rows at `relay_test.go:599-790`.
- `AwaitReparent` and `AwaitFirstRead`: `TestRelayWaits`.

CLI
- `login`, `setup`, pane refusals, `away` order, `stop`, `status`, `relay` verb: `TestSlackCLI` subtests.

Credentials, config, presence
- Token and setup files with owner-only modes, `RequireConfigured`, presence: `TestCredentialFiles`, `TestPresence`.
- `remind_after_min` loader: `TestLoadConfig`.

Hooks
- Dobby tick relay start: `TestHubDobbyDeps_PrimeTickEnsuresRelay`.
- Anchor change restarts the relay on every path: `TestChangeAnchor` row, subtests at `anchor_test.go`.
- Prime bring-up starts the relay, with warning on failure and nil for a beam: `startdobbies_test.go` and `start_test.go` rows.
- Resume pointer presence line: `resumecontext_test.go` rows.
- `ReadWait` and `ErrProgramClaimInStrandPane`: `creelcli/cli_test.go:1658` and `creelengine/identity_test.go:84`.
- `SlackGeometry`: `TestSlackGeometry`.
- Relay loops in the declared wakeup list: `cmd/lym/daemonwakeups_test.go:52`.

**Findings**

- **F-T1, MEDIUM, design.** `creelcli.HubStrands` (`internal/creelcli/goclient.go`, added by the diff) has no test. `slackcli/wiring.go` `hubPaneCaller` calls it, and the pane refusals depend on it. A grep across all `_test.go` files found no `HubStrands`, and no test of the exported `PaneOf` or `IsOrchStrand` wrappers either. `TestSlackCLI` drives `paneCaller` through fakes only. Fix: add a row to `creelcli/cli_test.go` asserting `HubStrands` lists the same strand set the daemon resolves.
- **F-T2, LOW, design.** Production wiring in `slackcli/wiring.go` has no test: `wireProduction`, `beamSlugs`, `orchPaused` and `receiverPaused`. `receiverPaused` picks the prime's pause for the hub orch and the beam's own for a beam orch. A wrong worktree choice would show a wrong "hub is paused" answer, and no row would fail. Fix: add one `receiverPaused` row per receiver kind, or state the gap.
- **F-T3, LOW, design.** `StopRelay` and `RelayLive` have no test that reaches `runproc.EndRole`. `TestSlackCLI` fakes `stop`, and `RelayLive` is only reached through `TestEnsureRelay` with real locks. I found no row that registers a relay record and checks that stop ends it and spares the caller and its ancestors. Fix: add a `StopRelay` row over a registered record.
- **F-T4, LOW, design.** D5 says an over-cap message is counted as handled "whether that line posts or not". `MessageSlackRefusesFiveTimesIsReplacedByOneLine` (`outbound_test.go:119-139`) covers only the success path of the replacement line. Nothing fails if the drop were skipped when the line's post fails. Fix: add a row where the sixth `PostMessage`, the replacement line, also fails, and assert the read is still acknowledged.
- **F-T5, NIT, design.** `ensureSlackRelay` (`beamcli/dobbydeps.go`) logs a gave-up start once per episode. `TestHubDobbyDeps_PrimeTickEnsuresRelay` writes `start_failures: 5`, so it probably covers the gave-up path, but I did not read its assertions. Confirm it asserts that the tick returns no error and that the log is not repeated.

I found no new test that duplicates another's coverage. The relay tests for the restart cap each pin a distinct lifetime case.

**Tool calls**
1. `grep` over the slack test files for test and row names. My context held no test files.
2. `grep` for `t.Run` and row names across the same files. Needed the subtest names inside the large tests.
3. `git diff` and `grep` for test names in the hook-package tests, plus `has_more`, `ProgramClaimInStrandPane` and the daemon-wakeups entry. My context held no test diffs.
4. `grep` for `HubStrands`, `receiverPaused` and the other production-wiring symbols in test files, plus `sed` of `outbound_test.go` lines 119-140. Checked the untested-symbol candidates and the refusal-line row.
