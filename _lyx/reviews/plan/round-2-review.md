---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-draft-writers.md:3
    summary: "Round 1 F2 recurs: the forced edits outside a card are still found by hand, and round 2 finds more sites in cards 4, 7, 13, 17, 27 and 36 where an existing test, allowlist or comment breaks and no card lists it"
    fix: "Apply these per-card additions, then, for every other card that deletes, re-signs or changes the observable behaviour of a symbol, grep the repository tests and comments for readers of the old behaviour before handing off. (a) Card 7: add Edit targets `internal/battencli#TestCancelIntegration_BlockedRunWithNoPair`, `internal/battencli#TestCancelIntegration_KeptPairJudgeOnly`, `internal/battencli#TestCancelIntegration_RemoveTearsDownPairAndBranch` and `internal/pairteardown_test#TestRun_TeardownScenario`, and say in the Tests paragraph that the three cancel tests (cancel_integration_test.go lines 125, 196, 256) and the `BoardClaimIsClearedWhenBattenNeverSeededTheRun` row (teardown_integration_test.go line 391) assert `draft` where they assert a nil status today. (b) Card 13: add Edit targets `internal/battenshed#TestStuckReport_LevelFollowsCause` and `internal/battenshed#TestNewInnerRun_FloorsPollInterval`, and say in the Tests paragraph that the two InnerRun rows of the first (`child moves from running to awaiting`, driven by `clock.onSleep`, and `blocked child's wait ends on a pause request`, driven by `clock.pauseAtSleep`) drive successive Calls, the first yielding; the floor test is settled by F2. (c) Card 4 line 43: add `gopkg.in/yaml.v3` to the card's own imports, since `LoadConfig` decodes the resolved document as `internal/battenshed`'s `LoadConfig` does. (d) Card 27, the `Live` and `SpawnDetached` paragraph: say `SpawnDetached` detaches the child through `proc.Detach`, as `fabricengine` does, so `dobbyengineAllowedImports` gains `internal/proc` beside `internal/lock`, and drop the words `the one import this card brings in`. (e) Card 36, the `TidySpec` paragraph: add `internal/parentdirective`, `internal/editdirective` and `internal/stencil` to the imports the card brings in, for the two directives and the stencil fill `TidyPrompt` makes as `loomcli`'s arbiter prompt does. (f) Card 17: add Edit target `//contracts/recipes/batten-recipe.yaml` and say its header comment's Land paragraph drops `and names the tasks the landing made ready` and states that a records stop is the one notice of a successful landing. Checked with `lyx loom validate-plan --overlay` over the target additions of (a), (b) and (f): no blocking finding."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/13-innerrun-short-check.md:106
    summary: "After cards 13 and 14 nothing sleeps in InnerRun, yet InnerRunDeps.Sleep, the poll interval and the running-arm reason text saying it checks every poll interval all stay"
    fix: "In card 13 state the fate of the poll cadence. (1) Add Edit target `internal/battenshed#innerRunProducer.runningStep` and rewrite its two reason texts to name no cadence, for example `inner shed run still running; dobby re-checks it on each probe or child status write`, with `TestInnerRun_RunningArmReviewWaitNote` (already listed) asserting the new text. (2) Delete `InnerRunDeps.Sleep` with its `waitOrCancel` default in `NewInnerRun` and its `InnerRun.Sleep` entry in `envkit`'s `nilLegal` (already listed); add Edit targets `internal/shedrecipe#innerRunEntry` (its doc names `Env.InnerRun.Sleep`) and `internal/shedrecipe#TestInnerRunEntry_NilSleepIsAccepted` (drop the Sleep assertion); the `InnerRun.Sleep` assignments in `wireForHub` and the two rewritten steps are already listed. (3) Say that `poll_interval_s` stays only as a validated key whose one effect is the sub-second floor Warn, and keep `TestNewInnerRun_FloorsPollInterval` as the Warn-only rows. Checked with `lyx loom validate-plan --overlay`: no blocking finding."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/33-dobby-passes.md:10
    summary: "Card 33 gives Deps.SpawnPass the result type PassExit, an exported type that no card creates and that card 37 must fill"
    fix: "In card 33 add under Create `plan:internal/dobbyengine#PassExit` -> `type PassExit struct`, carrying the exit code and the last error text, and add `plan:internal/dobbyengine#PassExit` to card 37's Uses. Checked with `lyx loom validate-plan --overlay`: no blocking finding."
---

### [MEDIUM] Forced edits outside a card are still found by hand and still missed

**Location:** `_lyx/plan/07-draft-writers.md:3`, `_lyx/plan/13-innerrun-short-check.md:121`, `_lyx/plan/04-dobby-records.md:43`, `_lyx/plan/27-dobby-start.md:24`, `_lyx/plan/36-dobby-board-tidy.md:32`, `_lyx/plan/17-land-drops-ready.md:3`
**Issue:** Round 1 F2 listed the tests, callers and allowlists that cards 13, 28, 4, 35 and 36 force outside their own packages, and the round-1 fixes are in the plan.
The same method still misses sites, each verified against the tree:
- Card 7 writes `draft` where cancel and the claim settlement wrote a nil status. The three `TestCancelIntegration_*` tests (tier1) assert `status != nil` is an error after a cancel, and the `BoardClaimIsClearedWhenBattenNeverSeededTheRun` row of `TestRun_TeardownScenario` (tier2, package `pairteardown_test`) asserts the same after `Run`. All four read `draft` after the card and fail. Card 7 lists only the unit-level `TestClearRunClaim` and `TestTeardown_CallSequence`.
- Card 13 turns the in-call wait into one check. `TestStuckReport_LevelFollowsCause` has two InnerRun rows that need a sleep to end the Call (the fake clock's status bump and its `pauseAtSleep` both fire from `Sleep`), and `TestNewInnerRun_FloorsPollInterval` asserts the Call slept exactly once. With `once` both rows yield and the floor test sees no sleep. All sit in the package the card edits and run in the card's tier0 gate.
- `scankit.AssertImportAllowlistNoStale` fails on a missing import as on a stale one, and three cards state their list as complete when it is not. Card 4's `LoadConfig` decodes bytes that `configengine.Load` returns into `Config`, so the package imports `gopkg.in/yaml.v3` as `internal/battenshed/config.go` does. Card 27's `SpawnDetached` detaches a child, and the owner of that is `proc.Detach` (`internal/fabricengine/spawn.go`). Card 36's `TidyPrompt` fills a stencil and renders the parent and edit directives, which `internal/loomcli/arbiterprompt.go` does through `parentdirective`, `editdirective` and `stencil`.
- Card 17 removes the ready part of Land, and `contracts/recipes/batten-recipe.yaml:26` still says Land "names the tasks the landing made ready"; card 13 edits that header for Run-Shed only.

### [LOW] InnerRun's poll interval and Sleep seam survive with no reader

**Location:** `_lyx/plan/13-innerrun-short-check.md:106`; `internal/battenshed/innerrun.go:680`, `:784` and `internal/battenshed/innerrun_pause.go:58`.
**Issue:** `deps.Sleep` is read in two places today, the `wait` loop that card 13 deletes and the spawn hold that card 14 turns into a yield.
After both cards `InnerRunDeps.Sleep` has no reader, `pollInterval` is read only by the reason texts `checking it every %s` (which a pass that checks once, scheduled by dobby on a status write or a probe, no longer makes true), and `poll_interval_s` in the recipe stays a live key with a floor Warn and a test.
Neither card says so; card 13 only rewrites the tier1 steps that used `InnerRun.Sleep` as a test hook.
Keeping a seam nothing reads is the dead code `code-quality` forbids, and the operator-visible reason in `lyx batten status` would state a cadence that is false.

### [LOW] PassExit is used by two cards and created by none

**Location:** `_lyx/plan/33-dobby-passes.md:27`
**Issue:** Card 33 describes `SpawnPass(slug string, continuePass, noReopen bool) (<-chan PassExit, error)` as a `Deps` field, and card 37's `hubDobbyDeps` fills it, but no Create list holds `PassExit`.
It is an exported type of the engine's seam surface, so it needs its own glyph for card 37 to name under Uses.

## Round 1 re-check

Every round-1 finding's fix is present in the plan and none has regressed: the prime-anchor working directory (cards 28 and 33), the listed forced edits of cards 13, 28, 4, 35 and 36, `begun` before `observeState` (card 13), the batten goto guard (card 25), the undecodable-record reads (cards 4, 31 and 32), the re-sign arrows (card 23), the covered owner check (card 14), the pause-honoured hand-back (card 24), the local-Bolt owner read (card 34), the dobby verb's bound (card 37), the dropped landing-halt row (card 24), the pre-run claim row (card 26), the owner read in `ensureParentDobby` (card 28), the release step in the refusal texts (card 7), `--keep-local` for a live pair (card 39), the failed-revive text (card 15), the carried tidy slugs (card 36), the stale docs (cards 4, 13, 22 and 28), the tidy spec's segment and skills (card 36) and the failed prime claim (card 27).
The round-1 fixer report holds nothing disputed and nothing deferred.

## Rejected

- Directive 1 (working directory of a pass): not a defect. Cards 28 and 33 start the pass with the prime's anchor as its working directory, which `lyxcwd.Resolve` accepts as the exact anchored directory of a git worktree, and batten's pre-run refuses a non-prime location anyway; the hub root is the dobby's own working directory only, which `lyx beam dobby` never resolves. The callers of `StartFirstPass` (`lyx adopt`, `lyx beam land`) pass the prime today.
- Directive 2, card 28: the removed `windowFlag`, `openTerminalFlag`, `windowOpener` and `terminalLauncher` have no remaining user outside the listed tests and docs.
  Card 35's allowlist entry (`internal/boardengine`) and card 13's (`internal/state`) are right and complete; the gaps in cards 4, 27 and 36 are in F1.
- Directive 3 (card 13, `begun`): not a defect. `begun` is read from the persisted episode state before `observeState` replaces it, `observeState` and `saveMemory` run on the same Call, and the next Call therefore starts from `done`; a child that goes running to done between passes gives the `child running → done` Stuck before the Done arm, and the closing test row pins that.
- Directive 4, card 25: the card states its bound (`GotoGuard` refuses every target but the halted row, with the refusal text), and a test row reaches it. Card 4 reads an undecodable dobby state as absent with one Warn, card 31's `dobbyReading` and card 30's `refuseContinuePass` read that as no dobby and a full pass owed, and card 32 reads the fresh record as a start.
- Directive 5, cards 24 and 18: a blocked pause-honoured Land stop persists as a blocked stop with the reason as its error, goes to the tackler by the rule table, and so can reach the arbiter; card 24's refusal and its `Check` row reach it. Batten has no landing halt through `landingshed.HandsBack`, and the card's test has no landing-halt row to be unreachable.
