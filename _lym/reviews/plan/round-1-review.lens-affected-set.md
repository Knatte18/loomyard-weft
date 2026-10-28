```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. For each output, string or behaviour a card changes, I listed what reads or asserts it and which cards cover it.

**Per change: the set found and the cards that cover it**

- **Hold-notice line (card 3).** Readers are `hold_test.go` `TestWait_HeldTurnEndNotice`, `attach_asking_test.go` `TestAttach_LegacyAskingAndHeldRuns`, `shuttleengine/doc.go:99`, the `holdNoticeSettle` comment, and `orch-template-role.md:243-244`. Covered by cards 3 and 12.
- **`ErrNoRunningTurn` and Inject refusal text (card 15).** Readers are `run_test.go:1023-1028`, which sits inside `TestRun_Interrupt_PlaysEscape`, and `run_inject_test.go:91`, inside `TestRunner_Inject_RefusesAnyInterruptKey`. Covered by card 15.
- **Removing `lym shuttle send` and `lym shuttle interrupt` (card 15).** The set is `send.go`, `interrupt.go`, `cli_test.go:222`, and `cli_test.go:604/607`, which sit inside `TestInterruptAndSendCmd_FailWithTheGuardsMessage`, a deleted test. It also takes in `messagetail.go:1,7`, the `run.go:1035` comment in `sweepOrphansOpportunistic`, `interruptguard.go:15,131`, `claudeengine/command.go:294`, `creelengine/doc.go:259`, `pattern/PATTERN-pane-text-injection.md:40,64,68`, `docs/overview.md:413` and `tools/sandbox/SANDBOX-SHUTTLE-SUITE.md`. `plugins/` names neither verb. Covered by card 15.
- **`post` form refusal text (card 2).** No test asserts either string; only `creelcli/cli.go:342,344` hold them. Covered by card 2.
- **`RequirementPeerMessaging` (card 13).** The set is `engine.go:185-186`, `requires.go:69`, `claudeengine/capabilities.go:25`, `capabilities_test.go:30`, `requires_test.go:37,86-88,127-132,196` (all inside `newRequiresRig` or `TestRunner_Requires`), `seatengine/prompt.go:162-167`, `prompt_test.go:180-186` (inside `TestSeatPrompts_ValuesPromptsAndSpecs`), `seatengine/table.go:82` (`Seat`) and `seatengine/doc.go:3,13`. No `.md` or `.yaml` file names it. Covered by card 13.
- **Creel trust sentence (card 14).** `prepare_test.go:325` and `settings_test.go:768` compare against the constant itself. `claudeengine/doc.go:217`, `command.go:338` and `claudeengine.go:226` name the sentence without quoting it. Covered.
- **Every `SendMessage` site (cards 3, 12, 16).** Go: `hold.go`, `shuttleengine/doc.go`, `orchengine/doc.go`. Stencils and testdata: all block 7 sites, the darn and webster asks, and both seat smoke fixtures. Covered, with the webster fork line allowlisted. Leftovers are in `_test.go` files only: `smoke_seats_test.go`, `smoke_cycle_test.go`, `hold_test.go` and `attach_asking_test.go` (see finding 3).
- **`MessageLine` and `LineFits` measuring the marked form (card 5).** Callers are `creelengine/verbs.go:35,44` and `loomcli/decision.go:304,312` (`noticeFits`). The creelengine callers are covered by card 5; `noticeFits` is not (finding 1).
- **New journal events `chased`, `chase-aged` and `unanswered` (card 7).** Log consumers outside creel are `orchcli/creelseam.go` and `tendercli/deps.go`. Both switch only on the events they know (`creelseam.go:138-154,173`, `deps.go:448-449`), so the new events change nothing there. Not affected.
- **Sweep runs on wider subjects (card 9).** Readers are `creelengine/doc.go` `# Removed holders` (card 9 covers it) and `pattern/PATTERN-daemon-wakeups.md`'s sweep bullet, which no card lists (finding 2).
- **Notifier wirings switching to `NotifyAsker` (card 4).** Only `orchcli/doc.go:147` describes the walk; the burler, loom and webster `doc.go` files do not. Covered by card 4.
- **Seat smoke fixtures moving to creel wording (card 12).** The fixtures' reader is `seatengine/smoke_seats_test.go` (tier3). No card lists it (finding 3).

**Findings**

1. **MEDIUM**, class consistency, card 5 (`05-expect-reply-engine.md`).
   - Card 5 makes `creelengine.LineFits` measure the longer marked line. That also tightens `internal/loomcli/decision.go#noticeFits`, which judges unmarked decision notices for both the strand and the `LOOM` program, so a `lym loom decision add` notice near the 8 KiB limit gets a different fit or inline answer.
   - The card neither lists that caller nor decides whether a notice that is never marked should be measured by the marked form.
   - Evidence: `internal/loomcli/decision.go:302-312`, `internal/creelengine/verbs.go:35`, card 5 Intent ("LineFits measures the marked form").

2. **LOW**, class consistency, card 9 (`09-sweep-holder-gone.md`).
   - Card 9 widens what the removed-holder sweep runs on, but `pattern/PATTERN-daemon-wakeups.md` still says the sweep "runs while mail or injects exist".
   - Card 9 does not list that file. Card 7 lists it only under `**Extends:**`, to add the chase timer, so neither card rewrites the sweep bullet.
   - Evidence: `pattern/PATTERN-daemon-wakeups.md`, the "creel's removed-holder sweep" bullet; card 7 Extends; card 9 Edit list.

3. **LOW**, class consistency, card 12 (`12-stencils-through-creel.md`).
   - After card 12 the chair asks with `lym creel post --expect-reply`, but `internal/seatengine/smoke_seats_test.go`, the test that drives those fixtures, still says the chair asks "over the session channel" and that "the chair's SendMessage starts a turn".
   - No card lists the file.
   - Evidence: `internal/seatengine/smoke_seats_test.go:3-4,125-126`, `internal/seatengine/testdata/smoke-chair.md:9`.

Tool calls:
1. Grepped for readers of the `post` form refusals, for `plugins/` and testdata naming the removed verbs, and for which test functions hold `run_test.go:1027` and `cli_test.go:604`: the pre-read lacked their callers and enclosing functions.
2. Grepped the burler, loom, webster and orch `doc.go` files and `docs/overview.md` for the notifier walk, and Go and markdown files for the trust-sentence wording: needed the docs that describe the changed behaviour.
3. Grepped tests and Go text for the stencil sentences card 12 rewrites: needed what pins the stencil wording.
4. Grepped for `LineFits` callers, creel log consumers outside creel, docs of the requirement list, and `cmd/lym` tests naming shuttle verbs: needed callers one hop beyond the pre-read.
5. Read `loomcli/decision.go:295-315`, loom tests on the fit limit, and the event handling in `tendercli/deps.go` and `orchcli/creelseam.go`: needed to see how the new `LineFits` caller and the log consumers use what changes.
6. Read `orchcli/creelseam.go:130-180`: needed to confirm that unknown log events fall through.
