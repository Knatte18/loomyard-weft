---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-orch-beam-up.md:21
    summary: "card 10 removes `lyx orch start` while the shipped `prime` shortcut still names it until card 12, so card 10 cannot pass its own tier0 gate"
    fix: >-
      Add `internal/shortcutcli/shortcuts.yaml`, `internal/shortcutcli#TestMount_File` and `cmd/lyx#TestShortcuts_FileAndRootListing` to card 10's **Edit:** list,
      and add one sentence to its Intent: the shipped `prime` shortcut's first argv moves from `orch start` to `beam start`, which exists until card 11 reworks it,
      with `TestMount_File`'s `prime` row and the cmd/lyx operator-index line reading `shortcut: lyx beam start, then lyx reed attach`.
      Card 12's Intent then says its yaml edit replaces the already repointed `prime` with `s` instead of dropping a line that names `orch start`.
      The alternative is to run card 12 before card 10 through `lyx loom remove-card` and `lyx loom insert-card`, which renumbers the cards and leaves every prose `card N` reference to fix by hand, so the repoint is the cheaper fix.
      Checked with `lyx loom validate-plan --overlay`: the three added targets add no blocking finding over the baseline.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/15-pawl-engine.md:71
    summary: "card 15 plans its tests as untagged, but PATTERN-test-speed requires every test file holding a Test function to carry exactly one tier tag"
    fix: >-
      In card 15's last Tests paragraph replace `every test is untagged with `t.Parallel`` by `every test file carries the `tier0` tag and every test calls `t.Parallel``,
      keeping `TestMain` in its own untagged support file as the entry says.
      `cmd/lyx/tiertags_test.go` refuses a test file with a Test function and no tier tag, so an implementer following the card as written fails that guard.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/16-pawl-recipe-rows.md:12
    summary: "card 16 adds `Env.Pawl` but lists none of the shared nil-seam fixtures that every Env section must join, so TestFullEnv_FillsEverySeam, TestNewPawl and both TestWire_EverySeamFilled fail"
    fix: >-
      Add `internal/testkit/envkit#FullEnv`, `internal/testkit/envkit#nilLegal`, `internal/loomcli#intentionallyNil` and `internal/battencli#intentionallyNil` to card 16's **Edit:** list, after `internal/shedrecipe#Env`.
      Add one Intent sentence: `FullEnv` fills every `Pawl` seam with a valid no-op so the recipe tests build the pawl rows, `nilLegal` lists `Pawl.Watch` since a nil watch leaves the timer alone,
      and both `intentionallyNil` tables list each remaining `Pawl.<Seam>` as a pawl-only seam, as they list the `BeamDescribe` seams today.
      Checked with `lyx loom validate-plan --overlay` together with F4's fix: no blocking finding beyond the baseline.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/14-finalize-land-recheck.md:11
    summary: "card 14 adds `LandBlockers` and `ParentWorktree` to `landingshed.Deps` but lists neither the reflective TestLandingDeps_EveryFieldPopulated nor the envkit nil-seam table"
    fix: >-
      Add `internal/testkit/envkit#nilLegal` and `internal/loomcli#TestLandingDeps_EveryFieldPopulated` to card 14's **Edit:** list, after `internal/landingshed#Deps`.
      Add one Intent sentence: `nilLegal` gains `Landing.LandBlockers`, a documented nil default, so `TestFullEnv_FillsEverySeam` and both wire tests keep passing,
      and the test's `unfilled` map gains `LandBlockers` and `ParentWorktree`, which a loom run leaves unset.
      Checked with `lyx loom validate-plan --overlay` together with F3's fix: no blocking finding beyond the baseline.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/15-pawl-engine.md:58
    summary: "card 15 adds an await marker and a dobby-down notice on re-entry that the decision record does not carry, and one test row posts a notice where an acceptance criterion says a Beam-Await re-entered at Beam-Await with no request posts none"
    fix: >-
      The parent records the await-marker re-entry in the decision record as an Added-after-Discussion entry under D9:
      a Beam-Await re-entered with its status naming Beam-Await, no request standing, the await marker written and the dobby not live posts one notice naming `lyx beam start <slug>` and `lyx beam land <slug>`,
      and the acceptance criterion's `posts none` then reads as the interrupted-landing notice only.
      No plan edit is needed once that entry stands, so the plan fixer defers this finding to the parent with that reason.
      The plan keeps the mechanism, since an earlier round added it to close the window between the accept's dobby stop and Done, and dropping it would reopen that window.
      If the parent declines, drop the `AwaitMarkerName` Create target of card 15 with the await marker, `DobbyLive` and the dobby-down row it carries, and the matching clauses of card 17 and the overview's last Shared Decisions bullet;
      dropping the `AwaitMarkerName` Create target adds no blocking finding over the baseline in `lyx loom validate-plan --overlay`, since no card lists it under Uses.
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/10-orch-beam-up.md:30
    summary: "card 10 deletes TestSmokeOrch_Adopt but leaves startPlainRun, killTreeProcs and tailString in smoke_live_test.go, which only that test used"
    fix: >-
      Add `internal/orchcli#liveFixture.startPlainRun`, `internal/orchcli#killTreeProcs` and `internal/orchcli#tailString` to card 10's **Delete:** list after `internal/orchcli#TestSmokeOrch_Adopt`.
      `waitForDiag` stays, since `TestSmokeOrch_CompactCycle` uses it.
      Checked with `lyx loom validate-plan --overlay`: no blocking finding beyond the baseline.
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/plan/16-pawl-recipe-rows.md:42
    summary: "card 16's Tests paragraph names TestNewBeam and TestBeamRouting, which card 6 renamed to TestNewPawl and TestPawlRouting"
    fix: >-
      In card 16's Tests paragraph write `TestNewPawl` and `TestPawlRouting` where it writes `TestNewBeam` and `TestBeamRouting`.
      The two matching **Edit:** glyphs are the plan gate's glyph-not-found check, which fixes them with the same rename.
---

### [BLOCKING] Card 10 breaks the shipped shortcut file until card 12

**Location:** `_lyx/plan/10-orch-beam-up.md:21`, `_lyx/plan/12-shortcut-s.md:15`
**Issue:** Card 10 deletes `orchCLI.startCmd`, so `lyx orch start` stops resolving.
`internal/shortcutcli/shortcuts.yaml` still carries `prime: [[orch, start], [reed, attach]]` until card 12 rewrites it.
`cmd/lyx/shortcuts_test.go`'s `TestShortcuts_FileAndRootListing` (tier0) asserts `shortcutProblems(root, shortcuts)` is empty, and its argv check refuses a shortcut whose argv no longer resolves to an invocable command.
`TestMount_File` in `internal/shortcutcli/shortcut_test.go` pins the shipped file's `prime` row to `shortcut: lyx orch start, then lyx reed attach`.
Card 10 itself edits `cmd/lyx#TestExitCode_HelpPaths` and `cmd/lyx#newRoot`, so the `cmd/lyx` test package runs in its card gate and the first test fails there.
Card 12 owns the yaml and both tests, so nothing in card 10 lets the implementer fix it inside its scope.

### [BLOCKING] Card 15 plans untagged tests against PATTERN-test-speed

**Location:** `_lyx/plan/15-pawl-engine.md:71`
**Issue:** `PATTERN-test-speed` says a test file holding a `Test` function carries exactly one `tierN` tag and an untagged test file holds support only, enforced by `cmd/lyx/tiertags_test.go`.
Card 15 says every test is untagged.
Cards 23 and 26 say tier0 for the same kind of test, so the card is the one that departs.

### [MEDIUM] Card 16 leaves the shared nil-seam fixtures out

**Location:** `_lyx/plan/16-pawl-recipe-rows.md:12`
**Issue:** `Env.Pawl` is a new seam section of `shedrecipe.Env`, and four existing places enumerate every `Env` section.
`internal/testkit/envkit` `FullEnv` fills each section, and `TestFullEnv_FillsEverySeam` requires `NilSeams(FullEnv(t))` empty, so an unfilled `Pawl` fails it.
The loomrecipe fixture `testEnv` starts from `FullEnv`, so card 16's `TestNewBeam` row asserting the nine rows cannot build `NewPawl` without filled `Pawl` seams.
`TestWire_EverySeamFilled` in `internal/loomcli/wiring_seams_test.go` and in `internal/battencli/wire_seams_test.go` fails on every `Pawl.<Seam>` they do not list, since both wire a loom or batten run that leaves `Pawl` unset; each already lists the `BeamDescribe` seams the same way.
None of the four is a target of card 16 or 17.

### [MEDIUM] Card 14 leaves the reflective Deps test and the envkit table out

**Location:** `_lyx/plan/14-finalize-land-recheck.md:11`
**Issue:** `TestLandingDeps_EveryFieldPopulated` in `internal/loomcli/landingdeps_test.go` walks every `landingshed.Deps` field of a loom run by reflection and fails on a zero value outside its `unfilled` map.
Card 14 adds `LandBlockers` and `ParentWorktree`, and card 17 leaves both unset for loom and darn, so the test fails from card 14 on.
`envkit.LandingDeps` leaves `LandBlockers` nil, so `TestFullEnv_FillsEverySeam` fails too unless `nilLegal` lists `Landing.LandBlockers`, as it lists the other optional landing seams.

### [LOW] Card 15 carries a re-entry the decision record does not carry

**Location:** `_lyx/plan/15-pawl-engine.md:58`
**Issue:** D9 names the re-entries of Beam-Await: a re-entry with no request and the run status naming Parent-Align posts the interrupted-landing notice, and a failed dobby restart posts a failure naming `lyx beam start`.
Card 15 adds a third, a re-entry with the status naming Beam-Await, no request and the dobby not live, told apart from a first entry by a new await marker file, which posts a dobby-down notice.
The acceptance criteria say a Beam-Await re-entered with its status naming Beam-Await and no request standing posts none, and the card's `TestBeamAwait` row posts one in that case.
The card also grows `Paths`, `Deps.DobbyLive` and Beam-Up's removal list for the marker, none of it in the record.
Round 1 added the mechanism for a real window, and round 2 accepted it, so the finding is about the record and not about the card's logic; the severity is LOW because the decision belongs to the parent.

### [LOW] Card 10 orphans live-smoke helpers

**Location:** `_lyx/plan/10-orch-beam-up.md:30`
**Issue:** `startPlainRun` (smoke_live_test.go:171), `killTreeProcs` (:449) and `tailString` (:593) are called only inside `TestSmokeOrch_Adopt`, which card 10 deletes.
They stay as unused code in a tier3 file.

### [LOW] Card 16 names the renamed tests

**Location:** `_lyx/plan/16-pawl-recipe-rows.md:42`
**Issue:** `internal/loomrecipe/pawlrecipe_test.go` holds `TestNewPawl` and `TestPawlRouting` since card 6.
The Tests paragraph still says `TestNewBeam` and `TestBeamRouting`.

## Rejected

- The two `glyph-not-found` blockers on card 16's `internal/loomrecipe#TestNewBeam` and `#TestBeamRouting` targets, which `lyx loom validate-plan --overlay '{}'` already reports: they are the plan gate's own check, so only the prose in F7 is raised.
- `TestRemovedVerbs_NamedNowhere` scanning `contracts/stencils`: it pins the acceptance criterion that no stencil or skill names a removed verb, as the other cmd/lyx scans do, and asserts no stencil's prose.
- The land-check halt in card 14 being arbiter-eligible, as the owner halt beside it is: the decision record words no hand-back for it.
- `TestNextAwaitWait` and `TestRunTakesNoSlot` over private helpers: no named test covers the doubling or the hold-to-slot mapping at their surface.
