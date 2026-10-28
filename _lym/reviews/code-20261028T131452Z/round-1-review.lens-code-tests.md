```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test row that would fail if it broke, or report none.

**Behaviour → pinning test**

- Sender quoting in `MessageLine`, the reminder and `read` text, EXTERNAL and programs included: `creelcli/readtext_test.go` rows, `creelengine/deliver_test.go` `reminderFor` and typed-line expectations. Pinned.
- Line cap at `post`, with the file way forward: `verbs_test.go` `PostInterrupt_RefusalsSendNoKeyAndStoreNothing` (LineOverTheBudget row) and the post-validation tests. Pinned.
- Reply to a strand refused over budget: `Reply_OverTheBudgetIsRefusedAndStoresNothing`. Pinned.
- Reply to EXTERNAL keeps the 64 KiB cap: the `verbs_test.go` comment "A reply to EXTERNAL is never claimed, so it keeps the 64 KiB cap". Pinned.
- Old-binary line past 10,000 UTF-16 units left out of every claim: `verbs_test.go:1046-1064` and `SkipsAMessageWhoseLineIsOverTheHookFieldCap`. Pinned.
- Old-binary line sets no flag: `flags_test.go` `OnlyAnOverlongMessageSetsNoFlag`. Pinned.
- Stop count (reset order, cap 6, unknown count at daemon start, `PostToolUse` reset): `verbs_test.go` `StopBlocksAreCappedAtSixPerTurn...` and `StopHoldsNoBlocksAfterADaemonStart...`. Pinned.
- Ring resets the count: `ARingStartsTheStopCountOverWhereTheDaemonStartCapRefusedMail` and `ARingAfterTheStopCapStartsAFreshCount`. Pinned.
- Ring (idle, a draft doesn't hold it, running turn, pause, hold, pane-mail never rung): the `TestScheduler_HookWake` rows. Pinned.
- NoTurn held from ring to a recorded turn end: `ATurnEndBeforeTheRing...` and `MailArrivingInAWokenTurnsBlockedChain...`. Pinned.
- 30 s unanswered-ring rule and the answered-by-claim and answered-by-running-turn cases: the three `ARing...` rows. Pinned.
- 5 s bell grace, draft hold, busy send, not-landed send, hold, pause, `CreelHooks` mark: the bell rows. Pinned.
- Waiter supersede: `ANewerWaiterOfTheStrandSupersedesTheOlder` and the cli superseded test. Pinned.
- Waiter reconnect across a daemon restart: `server_test.go` `HookWaitReconnectsAcrossARestart...`. Pinned.
- `hook-wait` exits 0 for a subagent, an unknown engine and a non-JSON payload: the cli hook-wait table. Pinned.
- `hook-read` exit rule and acknowledge only after the write: `TestHookReadExit`. The exit rule is pinned. Order is only indirectly pinned.
- Subagent filter in `hook-read`: the cli subagent test. Pinned.
- Merged Stop command (record or no record, flag absent, unwritable events file): `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`. Pinned.
- Waiter hook entry (asyncRewake, timeout 86400): `hookcmd_integration_test.go` and `settings_test.go` `Stop` rows. Pinned.
- `HookInput` parse (`agent_id`, `stop_hook_active`, `agent_type` alone): `TestClaude_HookInput`. Pinned.
- Block JSON with reason and systemMessage: `settings_test.go` `BoundaryBlock`. Pinned.
- Pane path narrowing (empty or unknown Provider keeps the pane path): the `delivery_test.go` Strands test and `StrandsWithoutPaneMailGetNoNoticeOrReminder...`. Pinned.
- `CreelHooks` set on launch and kept through reed resume: `settings_test.go` `Launch.CreelHooks`, `run_test.go`, `strand_test.go`, `strandops_integration_test.go`. Pinned.
- `post --interrupt` (order, no key on an invalid post, unmarked receiver, `ErrNoRunningTurn`, refused interrupt, store failure after the key, `--reply` refused): the `PostInterrupt_*` tests and the cli `--reply` refusal subtest. Pinned.
- Trust sentence file (deny notice, space, sentence verbatim): `prepare_test.go` and `settings_test.go`. Pinned.
- Launch and resume lines pass one `--append-system-prompt-file`: `command_test.go` rows. Pinned.
- Loom decision notice (split, refuse before recording): the three `decision_test.go` rows. Pinned.
- Help text carries the reply rule: the cli help subtest. Pinned.
- `Idle` returns `NoTurn` and `TurnEndedAt`: `delivery_test.go` rows and `session_test.go` rows. Pinned.
- Claim at the budget arrives whole in reason and systemMessage: the cli "whole lines up to the claim budget" subtest. Pinned.

**Findings**

1. **LOW, design.** The sentence's own init check, which refuses only line breaks, has no test (`claudeengine/settings.go` init, `creelTrustSentence` check). The file-content test only pins the sentence's text. Nothing shows the sentence may hold `"` while the deny notices still ban it.
2. **LOW, design.** `lym shuttle open` stays without the sentence (D9), and I found no test that pins this. My grep of the shuttleengine tests for the append-prompt file found only launch-line and `Prepare` tests. The session-open path has no assertion that it passes no `--append-system-prompt-file`.
3. **LOW, design.** `hook-wait` returns 0 when it cannot reach the daemon (`creelcli/hookwait.go`, `hubClient` error branch and `client.Call` error). The cli hook-wait table covers a subagent, an unknown engine and non-JSON input. It has no unreachable-daemon row, and no row for a reconnect budget that runs out. A change to exit 2 on that path would pass the tests.
4. **LOW, design.** A ring that writes the wake text to a failing stderr returns 0 (`hookwait.go`, the `Write` error branch). No row covers it, so changing it to return `code` would pass the tests.

**Test economy.** I found no clear duplicate, though I did not read the LLM-tier bodies in full. `SkipsAMessageWhoseLineIsOverTheHookFieldCap` and the `verbs_test.go:1046-1064` block both pin the 10,000-unit exclusion, so one may be redundant. I did not read either body in full, so I don't file it.

**Tool calls**
1. `git diff … -- '*_test.go'` filtered to function and row names. My context held no test diffs.
2. `git diff` of the session, reed, shuttleengines, readtext, prepare and settings tests, plus the `verbs_test.go` and `deliver_test.go` EXTERNAL and `stop_hook_active` lines, plus the `readiness.go` and `session.go` diffs. This checked what the rows assert.
3. `grep` of the shuttleengine tests for the append-prompt file near open or session, of `cli_test.go` for daemon-unreachable cases, and of the creelengine and creelcli tests for the 10,000 cap. This checked the gaps in findings 1 to 3.
