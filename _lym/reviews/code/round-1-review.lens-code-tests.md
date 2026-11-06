```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing each behaviour the diff adds or changes and the test that pins it. I read test names and some bodies only, and ran nothing.

**Per behaviour: pinning test, or none**

- **D1 registry** (`hubregistry`): `TestRecord` and `TestLookup` pin the upsert, the drop of a gone path or changed shortname, keeping an entry with no readable shortname or an unstattable path, and concurrent recorders. They also pin one match, none, several, a corrupt file and the `Live` ordering.
- **Registry directory** (`UserStateDir`): `TestUserStateDir`.
- **Daemon records its hub**: the `StartRecordsTheHubOnceAfterTheLockAndADaemonThatLostTheLockRecordsNothing` row, plus the record-interval and failed-record rows.
- **Hub directory seam**: `TestHubDirectory` pins the unresolved-versus-fault distinction, geometry, the `TrustedHubs` reload and the spawn pass-through.
- **D5 config**: `TestLoadConfig` pins the template and a hub value. `cmd/lym/configstrictness_test.go` (+1) and `internal/configreg/configreg_test.go` (+5) carry the registration. The local-layer refusal of `trusted_hubs` has no creel-specific row. It likely rests on the generic Guard check in `configreg_test.go` (lines 74–116), which I did not read in full.
- **D5 trust**: `TheStoredTrustFollowsTheOrchOfAListedHubOnLinuxAndTheListAtEachArrival` and `TheStoredTrustIsUntrustedForEveryOtherSenderAndForAFailedListAndAnotherPlatform`. The latter has an `orch-2` row.
- **D2 post-to-hub**:
  - `Post_ToAStrandOfAnotherHubIsJournaledOutboundAndWaitsOpen`.
  - `Post_ToAnotherHubIsRefusedWithItsWayForwardAndJournalsNothing`, with program and EXTERNAL senders and receivers, unknown, shared and unreadable shortnames, and the stamped 8 KiB cap.
  - `APendingOutboundMessageRaisesNoFlag`, and `APendingOutboundMessageMakesNoSweepSubjectAndNothingIsReturnedOrTyped`.
- **D8 refusals**: `AnotherHubsNameIsRefusedByEveryVerbThatStaysInsideOneHub`. `post --interrupt` is covered only if that row includes it, which I did not check.
- **D8 `hubs` verb**: the four `hubs` subtests in `cli_test.go` and the unreadable-registry rows.
- **D2 outbound worker** (`TestScheduler_Outbound`):
  - Delivery, down-then-spawn, refusal returning at once with the notice and the ended wait.
  - Return at the hour with the last transient reason, and the hour counted across a restart.
  - Backoff doubling, oldest-first order, and the in-flight deliver ending before the return.
  - The final call at the hour, the delivered-not-returned case, and restart survival.
  - The start cap of three, its reset, and the 200-byte rune cut with escapes.
  - Failed journal writes, the no-sweep-for-a-delivered-expects-reply message, and the 30-second timeout.
- **D3 deliver and confirm** (`TestDaemon_CrossHubDeliver`):
  - A forged sender being dropped, noticed and logged.
  - The flood and window rows, the duplicate deliver, the id clash, and the `no` cases for confirm.
  - A confirm that fails, times out or answers neither word.
  - The store lock not held across a confirm, overlapping deliveries, two hubs delivering to each other, and a strand with no known pane process.
  - Lookup faults, no `HubDirectory`, and a failed journal write.
- **D4 replies**: `Reply_ToAMessageFromAnotherHubIsJournaledOutbound…`, the 8 KiB refusal at `--reply`, `Forward_Of…IsRefused…`, and the outbound `ReplyFromTheHolder…` and `ADeliveredReplyNamingAnId…IsRefusedWithNoDrop` rows. Also `AReplyToAMessageFromAnotherHubStopsItsChase…` (no notice on settle) and `AGoneHoldersUnreadMessageFromAnotherHubIsJournaledUndelivered…`.
- **D6 stamps**:
  - The hook line: `ClaimedLineOfMailFromAnotherHubCarriesTheStampAfterTheSender…`.
  - The reminder: `InboundMailFromAnotherHubsStrandIsRemindedWithTheStampAndNeverTyped`.
  - Read text: the `MailFromAnotherHubPutsItsStampAfterTheSender` row in `readtext_test.go`.
  - Read JSON and the log: the "reads with trust and sending fields" subtest in `cli_test.go`.
  - `--wait-reply` reply JSON: `cli_test.go:533–554`.
- **Journal fold and retention**: the journal fold rows at `journal_test.go:224` and `:256`, and the retention carry at `journal_test.go:779`.
- **`reportengine`**: `replies_test.go` (+10).
- **Daemon wakeups declared list**: `daemonwakeups_test.go` (+1).
- **`tmuxkit.Main` pointing `XDG_STATE_HOME` at its run directory**: `tmuxkit_test.go` (+4/−2).

**Findings**

1. **MEDIUM, `design`.** The local-layer refusal of `trusted_hubs` has no row of its own.
   - Location: `internal/creelconfig/config_test.go` (only `TestLoadConfig`, lines 27–28 plus the template row).
   - Decision-record criterion: "D5: a local layer file holding `trusted_hubs` is refused". I did not find a test that writes a local file holding `trusted_hubs` and expects a refusal. `configreg_test.go` may cover it generically through `LocalDenied`.
   - If the generic row is missing for `creel`, the Guard classification is unpinned.

2. **LOW, `design`.** The `post --interrupt` refusal to another hub is pinned only if the `AnotherHubsNameIsRefusedByEveryVerbThatStaysInsideOneHub` row includes it.
   - Location: `internal/creelengine/verbs_test.go`; I did not read the row body.

Nothing else turned up without a pinning test. Two unchecked points: whether `journal_test.go:779` also pins that a settled outbound message is not carried, and that a pending outbound message older than the cutoff is.

**Tool calls (6):**
1. `grep` of test names in `crosshub_test.go` and `outbound_test.go`: my context held no test files.
2. Diff-filtered grep of test names across the other test files, for the same reason.
3. Diff of the `journal_test.go`, `replies_test.go`, `cmd`, `tmuxkit_test.go` and `flags_test.go` changes, plus a grep for `--wait-reply` and `trusted_hubs` in the CLI and config tests: to see the fold and JSON coverage.
4. Grep for journal fold names, a Guard local-layer test and the `orch-2` / goos rows: to check the D5 and fold coverage.
5. Grep for `Of: eventOutbound` / `eventInbound` in the tests: to look for retention-carry coverage.
6. Grep for `Retention` in `outbound_test.go` and `journal_test.go`: to confirm the retention test at `journal_test.go:779`.
