---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: "The relaunch shape follows the parent's added decision (a limit on the skill-load turn repeats the launch unchanged: a fresh launch starts fresh again, a resume resumes the same session id), not the proposed always-resume. readLoadTurn asks AccountSwitcher.SwitchAccount directly instead of going through switchAccountAtLimit, whose *heldTurnEnd a load turn does not have. The test row asserts the SwitchAccount call that records blocked_until, since the row runs on switchFakeEngine and card 13's test owns the record itself."
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: ruling
    reason: "The parent ruled to delete card 14's deferred-prompt sentence and its row; both are deleted, and D7 step 2's empty PromptLine stands."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed-differently
    reason: "battencli is engine-neutral and cannot read claude's account file to test for an entry, so PickAccount gains a keepEntry flag: the complete branch passes true and the engine keeps an existing entry. The wire test asserts the call with keepEntry true; TestClaude_PickAccount holds the entry and no-entry rows."
  - id: F13
    disposition: fixed
---

# Plan fix, round 1

All changes are in `_lym/plan`. `lym loom validate-plan` answers ok.

## Changes

- **F1** — card 3: `internal/battencli#primeShuttleRunner` is listed under Edit and switches to `shuttleengines.NewForWorktree(..., hubgeom.EngineWorktree(location))`, so the dobby's board-tidy agent runs under the beam's account. The ImpactSummary now counts nine launching members.
- **F2** (fixed differently) — card 14:
  - Edit gains `Run.readLoadTurn`, `Run.loadSkillTurn` and `Run.loadSkillsThenPrompt`.
  - Create gains `relaunchLaunch` with `relaunchContinue`, `relaunchFresh` and `relaunchRepeat`, plus `loadAccountSwitchedError`. `relaunchUnderAccount` now takes a `relaunchLaunch`.
  - A new Intent paragraph routes a usage-limited load turn end to `SwitchAccount` before the load hold counts it. On true, the load hold ends and the run relaunches as `relaunchRepeat`.
  - Step 2 now describes all three launch shapes.
  - The deferred-prompt row is replaced by a row that drives a usage-limited load turn end through `Start`.
  - After the self-check, the case runs ahead of both counting paths in `readLoadTurn`, the parser signal and the event read.
  - Card 16's calls are renamed to `relaunchFresh`, and card 13's cross-reference names the load hold.
- **F3** — card 2: `configreg#TestModules_EveryTemplateKeyClassified` is listed under Edit. Its claude case in `wantMachineKeys` gets `account_pins` and `accounts`. After the self-check, its `wantLocalDenied["claude"]` gets them too.
- **F4** — card 16:
  - Edit gains `readLoadTurn` and `loadSkillTurn`, and Create gains `loadRefusedResumeError`.
  - The start-path case now sits in `readLoadTurn`, ahead of both counting paths, and returns to `loadSkillsThenPrompt`, which runs `relaunchUnderAccount(relaunchFresh)`.
  - The start-path row asserts that one load is typed before the fresh relaunch.
- **F5** — card 5: `adoptcli#world.deps` is listed under Edit. It sets a `PickAccount` that records each slug.
- **F6** — each doc moves from Extends to Edit, and the Intent names the sentences it rewrites:
  - card 7: `claudeengine/doc.go` (the project directory, the plugins file, the config directory, and, after the self-check, the prober's and `SessionHolder`'s registry sentences) and `shuttleengines/doc.go` (built with no config directories).
  - card 8: `claudeengine/doc.go` (the no-transcript refusal and the resume check's registry).
  - card 11: `reedengine/doc.go` (resume relaunching or dropping a non-live strand).
  - card 12: `shuttleengine/doc.go` (the step stop's removable strands, the held-session branch and the carry-over's waiter-gone rule).
  - card 14: the PATTERN background's done-when paragraph gains the held-strand exception, and the overview's shared-decisions line names it.
  - The overview's PATTERN-changes bullet stays as it is, because validate-plan requires it to match the decision record.
- **F7** (ruling) — card 14: the deferred-prompt sentence and its row are deleted.
- **F8** — card 11: the Intent states what the hold skips and that only a stop removal, `ReleaseStrandHold`, `RelaunchStrand` or card 15's settle ends it.
- **F9** — card 4: the `TestResolveAccount` Create bullet and its rows sentence are removed. Card 6's `TestPrepare_ExportsTheResolvedAccount` gains the row for a removed or retired entry and the row for an engine told no worktree.
- **F10** — card 12: `shuttlecli#shuttleCLI.stateCmd` and `TestStateCmd_ListsRunningRunsAndNarrowsByStrand` are listed under Edit. Each run's map gains `relaunching`, and the test gains a row. After the self-check, the ImpactSummary names the verb.
- **F11** — card 12: the `strandRemovable` bullet names `removeSupersededStrands` and the untied branch of `StopStepRuns`.
- **F12** (fixed differently) — card 5: `PickAccount` takes `keepEntry bool`. The `complete` branch calls the pick with `keepEntry` true before `reconcileTaskPair`, and rows are added to both tests. After the self-check, the ImpactSummary names the third call.
- **F13** — card 7: `enterSession` passes `fabricengine.ConfigDirs(location)` of the watched session's location. The `configengine.Dirs{}` fallback is dropped.

## Self-check

- fix-claims, MEDIUM — the card 14 and card 16 load-turn cases missed the parser's `SessionSignalAPIErrorTurnEnd` counting path (`loadturn.go:212-221`). Fixed as proposed: both cases now run ahead of both counting paths.
- fix-affected-set, BLOCKING — the card 2 fix left `wantLocalDenied["claude"]` stale (`configreg_test.go:161`). Fixed as proposed.
- fix-consistency, LOW — card 13's cross-reference named only `Wait`. Fixed as proposed.
- fix-consistency, LOW — card 5's ImpactSummary missed the `complete` branch's call. Fixed as proposed.
- fix-consistency, LOW — card 12's ImpactSummary missed the state verb. Fixed as proposed.
- fix-proposal, LOW — the registry doc sentences belong in card 7, which widens `sessionHolder` and `processLiveness`. Fixed as proposed: they moved to card 7, and card 8 keeps the resume check's sentence.

## Disputed

None.

## Deferred

None.

## Rulings

- **F7** — the ruling asks to delete card 14's sentence about typing the deferred `PromptLine` in place of the continue line, and its row, so D7 step 2 stands. Both are deleted. I did not flag the ruling to the parent.
- The parent's added decision on F2 (a limit on the skill-load turn repeats the launch unchanged under the new account) was carried out in card 14's `relaunchRepeat` shape. F2 is recorded above as fixed differently.
