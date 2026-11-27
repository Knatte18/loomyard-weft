---
dispositions:
  - id: F1
    disposition: fixed-differently
    reason: "The hold is read by one new step, `fabricengine#recordsPushCeiling`, which both `PushPairAnchored` and `coalescePushSides` call. The proposal's `PushPairAnchored` alone misses the detached bypass, which reaches `coalescePushSides` through `DetachedPush` and `CoalescePushRequest` (internal/fabricengine/teardownlock.go:98, internal/fabricengine/coalesce.go:116). The `RecordsHold` request field is also dropped, since once the push reads the hold itself only tests would set that field (PATTERN-test-only-code)."
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed-differently
    reason: "The hold-aware ahead read lives in `fabricengine#RunCommitsAhead` (card 8), not in `NewDeps`' `Ahead` alone. The starter's `ifAhead` read (internal/runpusher/start.go) and the dobby's `StartIfAhead` also read it. A hold-aware `Ahead` alone would re-mark a held-only pair owed on every tick. Card 4 adds `Entry.Owed`, so cards 20 and 21 tell held entries from owed ones."
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: "Card 9's part, 'HeldRecordsCommit callers re-record before they push', is not carried out. The push step lives in `fabricengine`, and `fabricengine` cannot call `runpusher.RecordLanding`, since `runpusher` imports `fabricengine` (00-overview.md, the leaf-ledger decision). Instead, a pair push that finds the run's unrecorded `Lym-Landing` commit sends no records past origin (card 10). The parent pass re-records the landing (card 11), whose landing target `Ahead` now also reads unrecorded trailer commits."
  - id: F7
    disposition: fixed-differently
    reason: "`TestParentPush` is placed in `internal/loomcli`, not `internal/runpusher`. The wiring it must drive, `pusherDeps`, is unexported in `loomcli`, so a `runpusher` test cannot reach it. Card 17's `Start` seam is a recording wrapper around the real `runpusher.Start`. Card 12's Finalize rows that filled `LandLocally` and `ParentAheadByOwed` also move to a tier1 `landingDeps` test, so card 12 follows the rule card 13 now follows."
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed-differently
    reason: "The `lym beam pr` tests are tier0 over fake deps (internal/beamcli/pr_test.go:35), so a row there would fake `PushBeam`. The refusal is pinned instead by a new tier1 test, `TestHubLandDeps_PushBeamRefusesWhileTheBeamHoldsLandings`, over a hub fixture and the real `hubLandDeps`."
  - id: F11
    disposition: fixed
  - id: F12
    disposition: fixed
  - id: F13
    disposition: fixed
  - id: F14
    disposition: fixed-differently
    reason: "`landLocally` holds the pair's weft-side push lock, through a new `fabricengine.HoldPairPushLock`, rather than the pusher lock. Every pair push takes the push lock (internal/fabricengine/pushanchored.go:96-118). The pusher lock keeps out only the pusher, so the catch-up and `lym fabric push` could still push between the commit and the record."
  - id: F15
    disposition: fixed-differently
    reason: "The `lym fabric add` row is added as proposed. The board row instead drives the exported `boardengine.SpawnSync` in `TestSpawnSync_RegistersUnderTheHubOwner`. Its callers pass only the Bolt directory (internal/heddlecli/cli.go:173, internal/tendercli/deps.go:154). The board spawn's seam, `syncExecutable` (internal/boardengine/spawn.go:32), is out of reach from `fabricengine_test`, where `TestSpawnStatusPush` lives."
  - id: F16
    disposition: fixed
  - id: F17
    disposition: fixed
  - id: F18
    disposition: fixed
  - id: F19
    disposition: fixed
---

# Round 1 fixer-report

## Changes

- **F1, cards 9 and 10.** Card 9 adds `fabricengine#recordsPushCeiling`, the hold read that `PushPairAnchored` and `coalescePushSides` both call, so `lym fabric push` and its detached bypass are held like every other push.
  `CodeCeiling` is still set only by `PushRun`. The Bound states that `lym fabric push` in a beam pushes the code side whole while its records meet the hold.
  `TestPushRun_StopsShortOfAHeldRecordsCommit` gains rows for `lym fabric push` and its detached form. Fixed differently: see the frontmatter.
- **F2, cards 9 and 11.** Card 9 adds `RunPush.CodeCeiling`. `PushRun` passes it through and derives a ceiling only when it is empty.
  Card 11's Uses gain `RunPush` and `PushRun`, and step 5 sets `RunPush.CodeCeiling` to the judged tip. A test row pins a beam pushed to its landing tip.
- **F3, cards 6, 11 and 19.**
  - Card 6: `PruneLedger` keeps landing-holding entries and skips the board and landing keys, with a new `TestPruneLedger` row.
  - Card 11: `RecordLanding` writes `Worktree` and, for a beam parent, `Slug`. Step 2's drop writes `Slug` and `Worktree` on the pair entry it creates.
  - Card 19: adds `pairteardown.LandingStands`. It also reads true when the pair's own key holds other runs' landings. `RemovePair` removes only an entry holding no landings.
  - A `TestTeardown_CallSequence` row covers a beam pair holding a child's landing.
- **F4, cards 4, 5, 8, 20 and 21.**
  - Card 4: adds `Entry.Owed`. An entry holding only `HeldRecords` is held, not owed.
  - Card 8: `RunCommitsAhead` counts records only up to `<hold>^`, with a `TestRunCommitsAhead` row.
  - Card 5: a level pass clears a held-only entry's timing fields. A parent with nothing to push waits on its push-now mark. The unreachable `TestRun` row is replaced by a reachable one.
  - Cards 20 and 21: held-only entries are left out of the owed period, the all-clear and the bar, with new `TestPushSummary` and `TestSyncStatus` rows.
  - Fixed differently: see the frontmatter.
- **F5, card 11.**
  - Step 4 skips a landing commit of a dropped run, one whose pair entry holds `HeldRecords`.
  - Step 4 refuses with `lym loom resume` only while a new `RunPastFinalize` seam reads the run still at Finalize. A run past Finalize is re-recorded instead.
  - New `TestParentPush` rows cover both cases.
- **F6, cards 9, 10, 11, 12, 19 and the overview.**
  - Card 11: a pass re-records lost landings from their `Lym-Landing` trailer commits before step 1. The landing target's `Ahead` also reads such a commit.
  - Card 10: a pair push with no ledger hold sends no records past origin while the run's trailer commit stands on its parent.
  - Card 19: `LandingStands` reads the trailer as well as the ledger.
  - Card 12: the kill case names card 10's trailer read.
  - Overview: the trailer decision's rationale names the rebuild.
  - Tests: a `TestParentPush` row deletes the ledger, and `TestCommand_RemoveRefusesWhileALandingStands` gains a lost-ledger row.
  - Fixed differently: see the frontmatter.
- **F7, cards 11, 12, 13, 17 and 19.**
  - Card 11: `TestParentPush` becomes a tier1 `loomcli` test that drives `pusherDeps`' real wiring over a `fabrickit` hub with a bare origin.
  - Card 12: adds the tier1 `TestLandLocally` and `TestLandingDeps_LocalLanding`.
  - Card 13: adds the tier1 `TestLandingDeps_ParentPushState`, through `landingDeps`.
  - Card 17: `TestPushOwedForCatchUp` becomes tier1 over a pair fixture and the real push lock.
  - Card 19: every row writes its ledger entries through `pushledger.Update` or `RecordLanding`.
  - Each moved test states its seconds.
  - Fixed differently: see the frontmatter.
- **F8, card 4.** Adds `internal/pushledger/testmain_test.go` to Create, with the Intent line.
- **F9, cards 8, 9, 10 and 12.**
  - Card 12: deletes `ArchiveParentMerge` and `TestArchiveParentMerge`, and edits `internal/fabricengine/doc.go`.
  - Cards 8, 9 and 10: drop that doc from their Uses. Card 8's empty Uses field is removed; card 8 later gains a new Uses for F4.
- **F10, cards 6 and 9.**
  - Card 9: `hubLandDeps`' `PushBeam` refuses while the beam holds landings, with the way forward. Card 9 edits `internal/beamcli/doc.go` and the doc comments of `landDeps.PushBeam`, `prOpen` and `prUpdate`. A new tier1 test pins the refusal.
  - Card 6: drops `internal/beamcli/doc.go` from its Uses.
  - Fixed differently: see the frontmatter.
- **F11, card 7.** Adds `TestStartDetachedPush_RegistersTheChild` to Edit, with its new owner. `TestPushSpawnCommand`'s pair rows also change owner.
- **F12, cards 3 and 20.** `internal/dobbyengine/doc.go` moves to Edit in both, each card with its doc line.
- **F13, cards 5, 9, 15 and 19.** `internal/pairteardown/doc.go` moves to Edit in cards 9 and 19, with their doc lines. Cards 5 and 15 drop it from their Uses.
- **F14, cards 12 and 17.** `landLocally` holds the pair's push lock across the status commit and the record. Card 12 adds `fabricengine.HoldPairPushLock` and a `TestLandLocally` row. Fixed differently: see the frontmatter.
- **F15, card 7.** Adds a `lym fabric add` row to `TestSpawnStatusPush`. `TestSpawnSync_RegistersUnderTheHubOwner` now drives the exported `SpawnSync`. Fixed differently: see the frontmatter.
- **F16, card 23.** Adds `pattern/PATTERN-shed-driver.md` to Prosa, with its Intent line.
- **F17, card 19.** Adds `battencli#endRunPhase`, which reads `LandingStands` before `WriteIntent`. `TestCancelIntegration_KeptPairJudgeOnly` gains a row in which no intent is committed.
- **F18, overview.** The `Applies to:` line moves to the landing-entry decision.
- **F19, card 4.** `Update` records the clear time whenever the hub goes from no owed entry to one, as well as when the last owed entry goes, with `ledger_test.go` rows for both.

`lym loom validate-plan` reports `ok: true` after every change, with informational findings only.

## Self-check

- fix-claims, BLOCKING: the detached bypass skips `PushPairAnchored`. Fixed with my own change, the shared step `recordsPushCeiling` in cards 9 and 10.
- fix-claims, LOW: card 12's kill-case claim. Fixed as proposed: it names card 10's trailer read.
- fix-claims, LOW: no exported `gitkit` helper repoints an origin URL. Fixed as proposed: cards 11 and 17 say `git remote set-url` through `gitkit.Git`.
- fix-consistency MEDIUM and fix-proposal MEDIUM, merged: the starter's ahead read still counts held commits. Fixed with my own change: the hold-aware read moves into `RunCommitsAhead` (card 8).
- fix-consistency, MEDIUM: card 5's `TestRun` row cannot reach `NewDeps`. Fixed with my own change: the row now pins the level pass over a `HeldRecords` entry, and the read is pinned in `TestRunCommitsAhead`.
- fix-consistency, MEDIUM: card 12's rows still fill `LandLocally` and `ParentAheadByOwed` by hand. Fixed as proposed: they move to the tier1 `TestLandingDeps_LocalLanding`.
- fix-consistency, LOW: the overview's trailer rationale is stale. Fixed as proposed.
- fix-consistency, LOW: card 10's ImpactSummary is stale. Fixed as proposed.
- fix-consistency, LOW: card 11's widened landing `Ahead` has no test. Fixed as proposed, in the lost-ledger row of `TestParentPush`.
- fix-consistency, LOW: card 13 lacks `PushFailureHeldBack` in Uses. Fixed as proposed.
- fix-proposal, MEDIUM: `LandingStands` counts a dropped landing's commit. Fixed as proposed: a run whose pair entry holds `HeldRecords` is excluded, with a test row.
- fix-proposal, MEDIUM: the clear time is not refreshed after an earlier clear. Fixed as proposed: card 4 records it on every move from no owed entry to one, with a row.
- fix-proposal, LOW: a landing-holding entry whose worktree is gone stays forever. Fixed with my own change: card 20's summary names such an entry with its worktree gone, so the outage is visible. Recovering the pair is the operator's step.
- fix-affected-set, LOW: three beamcli doc comments are stale. Fixed as proposed: card 9 edits `landDeps`, `prOpen` and `prUpdate`.
- fix-affected-set, LOW: `TestPushSpawnCommand`'s owner rows. Fixed as proposed in card 7.

Every fork returned a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
