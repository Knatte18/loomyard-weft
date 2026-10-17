---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lyx/plan/10-orch-cycle-over-creel.md (PATTERN change paragraph) and _lyx/plan/08-drop-hub-notice-log.md (PATTERN changes paragraph)
    summary: The two declared PATTERN entry rewrites grow PATTERN.md by about 157 bytes, but the overview has only 4 bytes left under the headroom line, so loomshed's verify gate refuses the branch before any test runs, and no card makes room.
    fix: Make room for both rewrites in the card that already targets PATTERN.md last, card 10. Add to its Intent that PATTERN.md must end no more than 4 bytes above its base size, which is 15560 bytes today against pattern.HeadroomBytes of 15564. Card 10 meets that by cutting at least the combined growth of both entry rewrites from other entries' overview lines, moving the cut detail into those entries' background files, and lists each touched pattern/ background file under its Edit. Take the cut from entries whose meaning a shorter line still carries, such as the longest lines (PATTERN.md lines 70, 118, 33, 89, 74), and keep the two declared texts verbatim. If no meaning-preserving cut exists, defer to the operator, since the remedy would be to shorten the declared texts or raise the cap, and either one departs from the decision record's PATTERN changes.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lyx/plan/01-shuttle-event-time.md (sessionFakeEngine.ParseSessionSignals paragraph)
    summary: Card 1 sets Event on the in-package fake's bare ASK lines, which its ParseEvents never yields, and on every STOP line. That breaks the SessionSignal.Event contract and the card's own pairing, stops foldDialogAsks from latching the ASK rows in dialog_test.go, and shifts eventsFileActivity for every shuttleengine test built on sessionFakeEngine. The card plans only TestRunner_ReadEvents rows.
    fix: In card 1, make the stamp reading and the Event flag opt-in through a new sessionFakeEngine field (for example markEvents bool), set only by the new TestRunner_ReadEvents rows. With the field set, ParseSessionSignals reads @<time> stamps and sets Event only on the lines fakeEngine and waitingEngine ParseEvents yield (the STOP-colon, ASK-colon and WAIT-colon lines), never on a bare ASK. Every other test over the fake keeps today's readings. Add internal/shuttleengine#sessionFakeEngine to card 1's Edit list beside internal/shuttleengine#sessionFakeEngine.ParseSessionSignals (checked with validate-plan --overlay).
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:affected-set
    location: _lyx/plan/01-shuttle-event-time.md (shuttlefake paragraph and ImpactSummary)
    summary: Making shuttlefake.Engine a SessionSignalParser with a non-inert default moves every kit consumer onto the parser branches of eventsFileActivity and watchDialog. That contradicts the kit's inert-unless-scripted package doc and its TestEngine_InertDefaults, which exists although the card says the kit carries no test. The ImpactSummary names only the orch watcher.
    fix: In card 1, keep the default inert. With ParseSessionSignalsFn nil, ParseSessionSignals answers no signals with len(data) consumed, so eventsFileActivity reads modTime and watchDialog folds nothing, as today. The @<time> stamp reading becomes a helper a test sets as ParseSessionSignalsFn. Add internal/testkit/shuttlefake#TestEngine_InertDefaults to card 1's Edit with an assertion that the default answers no signals (checked with validate-plan --overlay). Drop the sentence that the kit carries no test of its own. Rewrite the ImpactSummary to say the kit's consumers keep their readings because the default stays inert.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:way-forward
    location: _lyx/plan/10-orch-cycle-over-creel.md (Steps and outcomes, sendStep; Restart)
    summary: A live watcher whose post or inject call fails after creel journaled it, such as on a non-waiting call that card 4 now times out, keeps a pending step with no id. Only a restarted watcher runs adoptPending, so the next tick can queue the same /clear, /compact or note request a second time.
    fix: In card 10, state that every tick finding a pending step with no id runs adoptPending before sending, after a failed call as well as after a restart. Add a TestWatcher_CreelFailuresHoldTheCycle row in which the fake creel journals an inject but answers the call with an error, and assert that the next tick adopts the journaled inject and sends nothing new.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lyx/plan/04-creel-withdraw-and-refusal-code.md (Call timeout paragraph)
    summary: Card 4 says CLI behavior is unchanged because postCmd and readCmd pass --timeout only when waiting. But injectCmd passes its timeout to CallWatching without --wait too (internal/creelcli/cli.go:377), so a non-waiting inject with --timeout becomes bounded.
    fix: Add internal/creelcli#injectCmd to card 4's Edit list (checked with validate-plan --overlay). Extend the Call timeout paragraph so injectCmd also passes its --timeout only when --wait is set, and zero otherwise.
  - id: F6
    severity: LOW
    class: scope
    origin: handler
    location: _lyx/plan/00-overview.md (## verify:)
    summary: Cards 8 and 10 edit PATTERN.md and two pattern/ background files, whose format, link and line-length checks run in internal/pattern (TestOverview_RealFilePassesCheck), but verify runs no internal/pattern tests.
    fix: Append ./internal/pattern to both lyx gate test lines of the verify section of 00-overview.md, the untagged one and the --tags integration one (checked with validate-plan --overlay).
  - id: F7
    severity: LOW
    class: decision
    origin: lens:way-forward
    location: _lyx/plan/11-orch-start-withdraws.md (first round, failed withdraw)
    summary: Some failed first withdraws have no workable next step under repair-what-the-error-names-then-rerun. Rerunning from the same non-orch strand pane, where creel refuses ORCHWATCH's program claim (identity.go:46-49), fails again. An unknown pane process (identity.go:108-109) names nothing the operator can repair.
    fix: In card 11, extend the way forward of the failed-first-withdraw refusal to name two more causes. For a program claim refused from a non-orch pane, run lyx orch start from the orch's pane or a terminal outside every strand pane. For an unknown pane process, restart or remove the strand the error names. Add a TestStart_Refusals row for the program-claim refusal that asserts that way forward.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/02-creel-orch-receiver.md (Store.Hold bullet)
    summary: The new program-only refusal of a hold on an orch strand names only the rule. Creel's sibling refusals name a next step, and an operator's lyx creel hold on the orch reaches this refusal as EXTERNAL.
    fix: In card 2's Store.Hold bullet, have the refusal text also name the way forward, posting mail with lyx creel post or cycling the orch with lyx orch distill or lyx orch refresh. Have the TestStore_InjectAndHold refusal rows assert the full text.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lyx/plan/10-orch-cycle-over-creel.md (Other strand, failures, removals; creelFailed)
    summary: creelFailed records a creel failure in State.Stuck, but card 10 never says when it clears. After a failed release at idle in particular, lyx orch status keeps showing a stale creel failure, against D5's until creel resolves the watcher again.
    fix: In card 10, state that the next successful creel call clears a creel-failure State.Stuck. Add a TestWatcher_CreelFailuresHoldTheCycle row in which a failed release at idle is followed by a tick whose creel calls succeed, and assert that Stuck is empty.
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lyx/plan/04-creel-withdraw-and-refusal-code.md (Tests, TestDaemon_Protocol rows)
    summary: The protocol row in which a strand claim naming a listed strand from outside every pane gets an error matching ErrStrandClaimOutsidePane is reachable only on Linux. Elsewhere resolveFromEnvironment signs the claim as the strand (identity.go:84-92), and the rig runs at runtime.GOOS, so the row fails off Linux.
    fix: In card 4's Tests paragraph, state that this TestDaemon_Protocol row runs on Linux only, gated as the existing Linux-only identity row in internal/creelengine/server_test.go is.
  - id: F11
    severity: LOW
    class: design
    origin: handler
    location: _lyx/plan/02-creel-orch-receiver.md (checkMayAct and deliverableReceiver bullets)
    summary: Card 2 removes creel's orch guard, so injects, interrupts and reminders now reach the orch, but its Intent carries no bound line. It names who may act, while D1's bound covers more, such as every such sender being a process of the operator's user behind the 0700 lyxtemp root, and the hold and command inject staying program-only.
    fix: Add a Bound line to card 2's Intent restating D1's bound. EXTERNAL senders, programs and other orch strands may now inject into, interrupt and remind an orch strand, as on any strand. Every such sender is a process of the operator's user behind the 0700 lyxtemp root. A hold, and card 3's command inject, on an orch strand stay refused to any sender but a program.
---

### [BLOCKING] The declared PATTERN changes cannot pass the overview headroom gate

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md` (PATTERN change paragraph) and `_lyx/plan/08-drop-hub-notice-log.md` (PATTERN changes paragraph)

**Issue:** `PATTERN.md` is 15560 bytes, and `pattern.HeadroomBytes` is 16384·95/100 = 15564 (`internal/pattern/check.go:21-24`).
`patternlint.Lint`, run by loomshed's verify gate before any verify command (`internal/loomshed/verifygate.go:110-116`), refuses a branch that grows the overview past `max(base, HeadroomBytes)`.
Its way forward is to cut the named bytes from `PATTERN.md`, moving detail into the entries' background files (`internal/patternlint/lint.go:21`).
Card 10's single-writer entry line grows from 188 to 272 bytes.
Card 8's durable-state entry line grows from 185 to 258 bytes.
Each change alone overshoots the 4 bytes of room, and together they add about 157.
No card cuts anything from `PATTERN.md`, so whichever batch carries the first rewrite fails its verify gate with no test run.
The decision record fixes both texts, so the room has to come from other entries; which entries to trim is a choice the plan has not made.

### [MEDIUM] Card 1's Event flag on the in-package fake breaks the fake's contract and existing tests

**Location:** `_lyx/plan/01-shuttle-event-time.md` (`sessionFakeEngine.ParseSessionSignals` paragraph)

**Issue:** `sessionFakeEngine.ParseSessionSignals` reads a bare `ASK` line as an ask (`internal/shuttleengine/fakes_test.go:815`).
`fakeEngine.ParseEvents` yields an event only for `ASK:` (`fakes_test.go:381`), so an `Event`-flagged bare `ASK` contradicts `SessionSignal.Event`, which `sessionsignal.go:66` defines as "ParseEvents also yields an event from the same line".
It also unbalances the card's own one-to-one pairing, which then zeroes every time in the read.
`foldDialogAsks` skips asks whose `Event` is set (`internal/shuttleengine/dialog.go:88`), so the `"ASK\n"` rows of `dialog_test.go:87-99` would no longer latch.
Setting `Event` on STOP lines also turns `eventsFileActivity`'s answer for the fake's unstamped Stops from the zero time into `modTime` (`activity.go:150-163`).
That shifts the readings of the other shuttleengine tests built on this fake.
The card plans rows only for `TestRunner_ReadEvents` and lists none of the affected tests.

### [MEDIUM] Card 1's shuttlefake parser is not inert and reaches every consumer of the kit

**Location:** `_lyx/plan/01-shuttle-event-time.md` (shuttlefake paragraph and ImpactSummary)

**Issue:** With `ParseSessionSignalsFn` nil, card 1 has `shuttlefake.Engine` parse stamps and Stops by default.
Every consumer of the kit then takes the parser branches of `eventsFileActivity` (`internal/shuttleengine/activity.go:140`) and `watchDialog` (`dialog.go:47`).
Those consumers are burlercli, burlerengine, creelcli, loomcli, orchcli, seatengine, shuttlecli, claudeengine, tendercli, treadleengine and websterengine.
Six of those packages are not in `## verify:`.
The default also contradicts the kit's package doc, "Engine is inert until a field or func override scripts it" (`internal/testkit/shuttlefake/shuttlefake.go:3`).
The card says "as a test kit it carries no test of its own", but `TestEngine_InertDefaults` exists (`shuttlefake_test.go:14`).
The lens review's dispute BC19 rests on the same false premise.
The default `ParseEvents` returns the fixed `Events` field and ignores the data (`shuttlefake.go:136-145`), so "each line ParseEvents would yield as a Stop" has no meaning for the default fake either.
The ImpactSummary's "ReadEvents' only production consumer is the orch watcher" misses these effects.

### [MEDIUM] A failed step call in a live watcher can send the step twice

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md` (Steps and outcomes, `sendStep`; Restart)

**Issue:** `sendStep` persists the step as pending with no id before the call.
Only "a restarted watcher holding a pending step with no id" runs `adoptPending`.
A non-waiting creel call is not resumed after a dropped connection (`internal/creelengine/client.go:151-153`), and card 4 makes a non-waiting call return `ErrWaitTimeout` while the daemon may already have journaled the request.
Card 10 states no next step for a live watcher in that state, so its next tick can queue the same `/clear` or `/compact`, or post a second note request whose id replaces the first.
D5's restart rule exists to prevent exactly this double send.

### [LOW] Card 4's unchanged-CLI claim misses injectCmd

**Location:** `_lyx/plan/04-creel-withdraw-and-refusal-code.md` (Call timeout paragraph)

**Issue:** `injectCmd` calls `client.CallWatching(req, timeout, …)` whatever `--wait` says (`internal/creelcli/cli.go:377`).
Once card 4 applies the timeout to every request, `lyx creel inject <r> <t> --timeout 5s` without `--wait` can end with the wait-timeout exit.
The card says no CLI call's behavior changes, but it lists only `postCmd` and `readCmd`.

### [LOW] Verify does not run internal/pattern, which checks the edited PATTERN files

**Location:** `_lyx/plan/00-overview.md` (`## verify:`)

**Issue:** Cards 8 and 10 edit `PATTERN.md`, `pattern/PATTERN-durable-vs-ephemeral-state.md` and `pattern/PATTERN-guarded-interrupt.md`.
`internal/pattern`'s `TestOverview_RealFilePassesCheck` (`internal/pattern/overview_test.go:11`) runs the format checker over the real files: the size cap, the entry line cap, links and orphan backgrounds.
No verify line runs that package.

### [LOW] The way forward for a failed first withdraw does not cover two of its causes

**Location:** `_lyx/plan/11-orch-start-withdraws.md` (first round, failed withdraw)

**Issue:** `lyx orch start` signs as `ORCHWATCH`.
Run from a non-orch strand pane, creel refuses that program claim (`internal/creelengine/identity.go:46-49`), and rerunning `start` from the same pane is refused again.
The sender-resolution refusal for an unknown pane process (`identity.go:108-109`) names no worktree or daemon to repair.
For both causes, "repair what the error names, then run `lyx orch start` again" does not progress.

### [LOW] The new program-only hold refusal names no way forward

**Location:** `_lyx/plan/02-creel-orch-receiver.md` (`Store.Hold` bullet)

**Issue:** Creel's sibling refusals name a next step, such as "send it mail with `lyx creel post`" (`internal/creelengine/verbs.go:543`).
The new refusal names only that a program may hold an orch strand, and the operator's own `lyx creel hold <orch>` reaches it as EXTERNAL.

### [LOW] Card 10 never says when a creel-failure stuck reason clears

**Location:** `_lyx/plan/10-orch-cycle-over-creel.md` (Other strand, failures, removals; `creelFailed`)

**Issue:** D5 shows a creel refusal "as State.Stuck until creel resolves the watcher again".
Card 10 records the failure through `creelFailed` but states no clearing rule.
After a failed release at the return to idle, no later step overwrites `Stuck`, so `lyx orch status` keeps a stale reason.

### [LOW] Card 4's outside-pane protocol row is reachable only on Linux

**Location:** `_lyx/plan/04-creel-withdraw-and-refusal-code.md` (Tests, `TestDaemon_Protocol` rows)

**Issue:** Off Linux, `resolveFromEnvironment` signs a strand claim of this hub as the strand and never refuses it (`internal/creelengine/identity.go:84-92`).
The protocol rig runs at `runtime.GOOS` (`internal/creelengine/server_test.go:76`).
The row asserting `errors.Is(err, ErrStrandClaimOutsidePane)` therefore fails off Linux unless it is gated, as the existing Linux-only row near `server_test.go:517` is.

### [LOW] Card 2 removes the orch guard with no bound line

**Location:** `_lyx/plan/02-creel-orch-receiver.md` (`checkMayAct` and `deliverableReceiver` bullets)

**Issue:** Card 2 drops `checkMayAct`'s refusal of every typing into an orch strand (`internal/creelengine/verbs.go:460-462`) and lets reminders reach orch strands.
Its Intent says who may now act, but carries no bound line.
D1 bounds the change: every such sender is a process of the operator's user behind the 0700 lyxtemp root, and holds and command injects stay program-only.
The rubric's attack-surface rule asks the introducing card to state that bound.

## Rejected

- guard-reach, card 7: the claim that `revivePending` reads pending forever because settle records carry no From or To. The fold fills every event's From and To from its inject before the log filter runs (`internal/creelengine/fold.go:392-396`), and `setOutcome` keeps the inject in the pool, so `log --from TENDER --to <orch>` returns the settle events.
