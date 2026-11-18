---
verdict: BLOCKING
prior:
  - id: F1
    status: resolved
    answer: "Card 3 now lists battencli primeShuttleRunner under Edit, switches it to NewForWorktree, and counts nine launching members."
  - id: F2
    status: resolved
    answer: "Card 14 adds a readLoadTurn case ahead of both counting paths and a Start-driven row asserting the SwitchAccount call; the relaunch shape follows the parent's added decision."
  - id: F3
    status: resolved
    answer: "Card 2 lists TestModules_EveryTemplateKeyClassified under Edit with the claude machine and local-denied key lists, which match the test's sorted shape."
  - id: F4
    status: resolved
    answer: "Card 16 moves the start-path case into readLoadTurn, ahead of both counting paths, and keeps the row asserting one load typed before the fresh relaunch."
  - id: F5
    status: resolved
    answer: "Card 5 lists adoptcli world.deps and states it sets a PickAccount recording each slug. The same defect in battencli is the new finding F14."
  - id: F6
    status: resolved
    answer: "Cards 7, 8, 11 and 12 list their doc files under Edit and name the sentences rewritten, and card 14 names the PATTERN done-when paragraph. The same defect in card 14 and card 16 for shuttleengine/doc.go is the new finding F16."
  - id: F7
    status: resolved
    answer: "Settled by the added entry titled Drop card 14's deferred-prompt rule (F7): card 14 no longer holds the deferred-PromptLine sentence or its row, and the continue relaunch keeps an empty PromptLine."
  - id: F8
    status: resolved
    answer: "Card 11 states what the hold lets through and the four events that end it."
  - id: F9
    status: resolved
    answer: "Card 4 no longer plans TestResolveAccount, and card 6's TestPrepare_ExportsTheResolvedAccount carries the two rows no other test had."
  - id: F10
    status: resolved
    answer: "Card 12 lists shuttleCLI.stateCmd and its test and adds the relaunching key."
  - id: F11
    status: resolved
    answer: "Card 12's strandRemovable bullet names removeSupersededStrands and StopStepRuns' untied branch."
  - id: F12
    status: resolved
    answer: "Accepted as fixed differently: the keepEntry flag keeps battencli engine-neutral, the complete branch passes true, and TestClaude_PickAccount holds the entry and no-entry rows."
  - id: F13
    status: resolved
    answer: "Card 7 passes fabricengine.ConfigDirs of the watched session's location in enterSession, which matches the code."
findings:
  - id: F14
    severity: BLOCKING
    class: scope
    location: _lym/plan/05-account-pick-at-making.md:46
    summary: "Card 5 sets the new battenCLI.pickAccount field only in Command, but the existing tier1 create tests build a battenCLI literal, call wire and run CreateWorktree through top.Add, so the closure calls a nil func and panics; no card lists those builders."
    fix: "In card 5, drop `internal/battencli#Command` from the Edit list and change the Wiring paragraph so `battenCLI.wire` fills `pickAccount` beside the other seams, a closure that builds the new pair's location, takes `hubgeom.EngineWorktree` of it and calls `shuttleengines.AccountKeeper(...).PickAccount(keepEntry)`, instead of `Command` setting it. The existing create tests, which build `&battenCLI{location: ...}` and call `wire` (`wireForHub` in lifecycle_integration_test.go, the beamfork integration test, and the CreateWorktree rows of wire_seams_test.go), then reach the real pick, which writes nothing on a hub with fewer than two accounts. State that `TestWire_CreateWorktreePicksTheNewWorktreesAccount` overrides the field after `wire`, as the owner-seam tests do, and asserts `wire` leaves it non-nil. Checked with `lym loom validate-plan --overlay`, no new blocking finding."
  - id: F15
    severity: BLOCKING
    class: design
    location: _lym/plan/14-in-run-relaunch.md:56
    summary: "Card 14 and card 16 run relaunchUnderAccount from loadSkillsThenPrompt on the start path but never say what a relaunch that finishes the run returns there, and launch reads a nil error as a started run, so a died or timed-out relaunch would hand Start a finalized run as started."
    fix: "In card 14's load-turn paragraph, state what loadSkillsThenPrompt returns when relaunchUnderAccount reports a finished run: a relaunch that ends the run died, timeout or api-error returns that Result with a non-nil error wrapping ErrNotStarted, the shape abandonStartup returns, so start neither hands back a finalized run nor skips its retry judgement; a relaunch that ends it done returns Result{} and nil, as abandonLoad does. State in card 16 that its loadRefusedResumeError path follows the same rule. Add a row to TestWait_UsageLimitSwitchRelaunchesInTheRun driven through Start: a usage-limited skill-load turn whose relaunch Prepare fails makes Start return an error wrapping ErrNotStarted and no run. No target changes, so no check constrains the fix."
  - id: F16
    severity: BLOCKING
    class: consistency
    location: _lym/plan/14-in-run-relaunch.md:32
    summary: "Cards 14 and 16 list internal/shuttleengine/doc.go under Extends, but the load-turn cases the round-1 fixes added falsify sentences in it about the usage-limit hold, the API-error re-send and the skill-load hold; a regression of the F2 and F4 fixes, raised at BLOCKING because an incremental round admits no lower new severity."
    fix: "In card 14 and card 16, move `internal/shuttleengine/doc.go` from the Extends list to the Edit list and name the sentences each rewrites. Card 14 rewrites the usage-limit paragraph that opens 'A held turn end stopped by the provider's usage limit sends no notice' (an engine with the switch relaunches the run under another account instead of holding), the re-send paragraph that opens 'A run whose Spec.APIErrorRetries is set re-sends such a turn' (a limited turn end the switch handles is neither counted nor re-sent), and the skill-load hold paragraph from 'A load turn that ends on an API error' through 'until a load turn ends' (a usage-limited load turn end the switch handles ends the hold uncounted and repeats the launch). Card 16 rewrites the same skill-load hold paragraph (a copied resume's first load turn end on an API error ends the hold and starts fresh) and the re-send paragraph (such a first turn end is neither counted nor re-sent). Checked with `lym loom validate-plan --overlay` together with the F14 fix, no new blocking finding."
---

### [BLOCKING] Card 5's pick seam is nil in the existing create tests

**Location:** `_lym/plan/05-account-pick-at-making.md:46`

**Issue:** Card 5 gives `battenCLI` a `pickAccount` field "set in `Command`" and has the `CreateWorktree` closure in `battenCLI.wire` call it after `top.Add` and in the `complete` branch.
The tier1 tests that drive that closure never go through `Command`: `wireForHub` in `lifecycle_integration_test.go` builds `&battenCLI{location: h.Location}` and calls `wire`, and `beamfork_integration_test.go`, `TestBattenIntegration_RunReconcilesTheHubBeforeAndThePairAfterCreating` and the `CreateWorktree` rows of `wire_seams_test.go` do the same.
Each reaches `top.Add` or the `complete` branch, so the closure calls a nil func and the test panics.
F5 fixed the same defect for `adoptDeps` through `world.deps`; the battencli builders are the same class and no card lists them.
This is the registry-and-shared-builder question of the focus directive.

### [BLOCKING] A relaunch that finishes the run has no return contract on the start path

**Location:** `_lym/plan/14-in-run-relaunch.md:56`

**Issue:** The F2 and F4 fixes run `relaunchUnderAccount` from `loadSkillsThenPrompt`, which `launch` calls before `Wait`.
`launch` treats a nil error from `loadSkillsThenPrompt` as a started run and returns `Result{}`; every failure there goes through `abandonStartup` or `abandonLoad`, which return a non-nil error wrapping `ErrNotStarted` so `start` can judge a retry.
`relaunchUnderAccount` instead finalizes `OutcomeDied` or a timeout itself and returns `(Result, true, nil)` for a finished relaunch.
Card 14 says only that `loadSkillsThenPrompt` "returns its result", and card 16's start-path case says nothing, so a relaunch whose `Prepare`, transcript check or startup wait fails would return a nil error and `Start` would hand back a finalized run as started.
No planned row drives a failing relaunch through `Start`; the died and timeout rows run through `Wait`.
Focus questions 2 and 3 settle as follows: both start-path cases are now reached, in `readLoadTurn` ahead of both counting paths, and the Start-driven row asserts the call that records `blocked_until`; this contract is what they leave open.

### [BLOCKING] Cards 14 and 16 falsify sentences in a doc they list under Extends

**Location:** `_lym/plan/14-in-run-relaunch.md:32`

**Issue:** `internal/shuttleengine/doc.go` is listed under Extends on both cards, which makes it add-only.
Its skill-load hold paragraph says a load turn ending on an API error "skips nothing" and the hold "goes on so until a load turn ends"; its usage-limit paragraph says the hold is unchanged and a limited turn end sends no notice; its re-send paragraph says a run with `APIErrorRetries` re-sends such a turn.
The load-turn cases the F2 and F4 fixes added to cards 14 and 16, and the switch case in `Wait`, make each of those false for a switched or refused-resume turn end.
F6 fixed the same defect for cards 7, 8, 11 and 12 and left these two cards, whose new text came later in that round, out.

## Rejected

- Launch wiring: every production site that builds engines and starts a session is a card 3 target, namely `loomCLI.wire`, `burlerCLI.wireHub`, `wireCLI`, `shuttlecli.Command`, `battenCLI.wire`, `primeShuttle`, `primeShuttleRunner`, `websterCLI.wireHub` and `newStrandRig`.
  The weaver, tender, cancel, parent-lock and activity-read sites only probe, send to or stop strands, and the standalone burler and webster builds have no hub, so no wiring gap is raised.
- `switchFakeEngine` is described as a `resumeFakeEngine` with `AccountSwitcher`, `UsageLimitReader` and `SessionSignalParser`, while the Start-driven load-turn row also needs a skill loader and the `TurnEndActivity` read.
  The fake is a Create target the implementer defines, and the gap is below BLOCKING, so an incremental round does not raise it.
- Card 14 does not say how `readLoadTurn`, which returns an event and a flag, hands the switch or refusal to `loadSkillTurn`, through `loadTurnWatch` or a changed result.
  It is below BLOCKING and is not raised in an incremental round.
- The registry checks hold: `configreg`'s `TestModules_EveryTemplateKeyClassified` is listed with correct sorted lists, `ReedOps` implementers other than `fakeReed` and `shuttlefake.Reed` embed `fakeReed` or are reed's engine, `shell.Shell` has two implementers, and `orchengine.Session` has two.
