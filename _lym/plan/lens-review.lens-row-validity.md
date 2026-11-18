```yaml
lens: row-validity
outcome: reported
```

Task: row-validity lens review of the plan's planned test rows and new tests.

**MEDIUM — Card 5, `TestWire_CreateWorktreePicksTheNewWorktreesAccount`: the success row cannot be reached on the setup it borrows.**
The card builds the test "on the setup `TestWire_CreateWorktreeClaimsTheBoardEntry` uses" and asserts that `pickAccount` runs once after a successful `top.Add`. That setup's own doc says: "Every row that passes the claim fails the closure on the absent fabric config, so no pair is created." So the closure never gets past `top.Add`. The "called once after success" assertion can never be met, and the "not when the add fails" row passes even with no pick wired at all.
Evidence: `internal/battencli/wire_seams_test.go:706-710`.
Fix: give the closure a told `Add` seam, or a hub with a fabric config (tier1), and list that seam on the card.

**MEDIUM — Card 5, `TestClaude_PickAccount` row "two `PickAccount` calls at once leave one entry and one increment": nondeterministic.**
The guard compares `Picked` against a time noted before the lock is taken. Two goroutines prove nothing unless the second notes its time before the first writes. With no clock seam and no hook to hold the lock, the row may run the calls one after the other and see two increments. Equal timestamps at clock resolution also defeat "later than". Either way the row is flaky, or it passes without ever reaching the guard.
Evidence: card 5's Intent ("notes the time … keeps the entry … when `Picked` … is later than the noted time"). No clock or lock seam is declared on `Claude` in card 3 or card 5.
Fix: add a clock seam and a hook between noting the time and taking the lock, and list both.

**MEDIUM — Card 14, `TestWait_UsageLimitSwitchRelaunchesInTheRun` row "a `StopStrand` from another handle between the mark and step 4": the fake cannot interleave.**
`fakeReed` has only scripted queues and call logs, with no callback. Card 11 gives the new ops only "records each call in its `CallLog` with a scripted error". No fake field lets a stop run between `EndStrandProcess` and `RelaunchStrand`, so the row cannot reach the "a stop made the record terminal → remove the strand instead of writing" branch.
Evidence: `internal/shuttleengine/fakes_test.go` (`fakeReed` struct: no hook fields). Card 11's description of the `fakeReed` methods.
Fix: card 11 or card 14 adds a hook field, such as `OnEndStrandProcess func()`, and lists it.

**MEDIUM — Card 12, `TestAttach_OutcomeDisposition` row: the table cannot express it.**
The table has only `outcome`, `includeOutcome` and `wantFound`. It always scripts `liveStatus("strand-1", "%1")` and seeds through `seedAttachRunOpts`, which has no relaunching mark. The row "relaunching record within its bound over a held, paneless strand" needs a per-row strand status and a `RelaunchingAt` seed. Neither the table fields nor the `seedAttachRun` / `seedAttachRunOpts` change is listed on the card.
Evidence: `internal/shuttleengine/attach_test.go:188-215`.

**LOW — Card 12, `TestStopStrand_UnrecordableStopRemovesNothing` is the wrong host for its new row.**
That table's rows are `corrupt` functions, and it asserts that `stopRecorded` errors and removes nothing. The planned row ("records stopped and removes the strand") is a success case with the opposite assertions, so it does not fit as a row. It belongs in a StopStrand success test, or in a new row shape the card states.
Evidence: `internal/shuttleengine/stop_test.go:134-175`.

**MEDIUM — Card 11: `ReleaseStrandHold` and `EndStrandProcess`'s "failure after the mark leaves the strand held" have no reed-level row.**
`TestEndAndRelaunchStrand_KeepTheGUID` covers end, resume skip, relaunch and remove-then-refuse. The real `ReleaseStrandHold` (clears the mark, and a later resume relaunches the strand) is reached only through the shuttle fake, so a wrong reed implementation passes every planned test.
Evidence: card 11's test paragraph.

**MEDIUM — Card 17, `TestWatcher_UsageLimitSwitchRestartsTheOrch`: nothing asserts the watcher carries on over the new strand.**
The rows assert only that `Resume` is called once and that no usage-limit wait is recorded. The Intent says the watcher "carries on over the new strand from the state the restart saved". No row checks that the next tick reads events from the new strand GUID and the reset cursor rather than the old ones, so a watcher stuck on the old strand passes.
Evidence: card 17's test paragraph. `fakeSession` (`internal/orchengine/watcher_test.go:27`) scripts per GUID, so the row is writable.

**LOW — Card 7, `TestRecordReaders` row: it claims more than it tests.**
The row says the session-end usage recorder and `tools/tokencount` "count" a session on a listed non-default account. The test only exercises the adapter's `LocateSessions` and `ReadSession`. The existing `TestRecordReaders` also checks shape against a stub `readingEngine` and holds no HOME or fake-directory setup, so the row needs its own HOME setup, which forbids `t.Parallel()`. The card states neither. D8's tokencount and session-end criterion is reached only indirectly.
Evidence: `internal/shuttleengines/shuttleengines_test.go:306-345`.

**LOW — Card 6, `TestEnvScrub_MatchesTmuxkitServerEnviron`: the added environ entry only checks that two lists agree.**
Adding `CLAUDE_CONFIG_DIR=/x` checks that `EnvScrub` and `tmuxkit.ServerEnviron` agree. It does not show that a spawned server lacks the variable: if both sides forgot the key, the test still passes. The D4 criterion ("a tmux server spawned from a process carrying `CLAUDE_CONFIG_DIR` carries none") has no row asserting that `Apply`'s output lacks the key.
Fix: assert it explicitly, or add a row to `TestEnvScrub_Apply` in `internal/reedengine/envscrub_test.go`.
Evidence: `internal/shuttleengines/shuttleengines_test.go:220-245` and `internal/testkit/tmuxkit/tmuxkit.go:534`.

**LOW — Card 10, test economy: `TestPrepare_StatusLineNamesTheLaunchedAccount` is a new top-level test over the same setup as card 6's `TestPrepare_ExportsTheResolvedAccount`.**
Both test the same `Prepare` with the same resolved-account fixtures. "Listed account gets `statusLine`, unlisted gets none" fits as rows or assertions of card 6's test.

**LOW — Card 9: `internal/claudecli` is a new test package, but no `testmain_test.go` is listed.**
`PATTERN-test-isolation` requires `tmuxkit.Main`. Also, `TestCommand`'s "refusal outside a hub" depends on the process working directory, which needs `t.Chdir` and rules out `t.Parallel`; the card does not state this.

My tool calls:
1. Grepped every existing test and fake the cards name, to confirm they exist and where. My context had only the cards' claims.
2. Read the claudeengine test names, `TestStopStrand_UnrecordableStopRemovesNothing`, `resumeFakeEngine` and `TestRecordReaders`, to check that the planned rows fit their hosts.
3. Read `TestWire_CreateWorktreeClaimsTheBoardEntry`, `TestStart_DeadStrandRemovedBeforeStart`, `TestEnvScrub_MatchesTmuxkitServerEnviron` and `tmuxkit.ServerEnviron`, to check the card 5, 17 and 6 setups.
4. Read the orch start harness's recorders, `fakeReed`'s fields and `TestAttach_OutcomeDisposition`'s table, to check the card 17, 14 and 12 rows.
