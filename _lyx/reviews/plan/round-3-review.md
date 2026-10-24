---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/14-guard-step-per-tier.md, _lyx/plan/20-batten-switch-entry-gate.md, _lyx/plan/13-testtiming-tiers.md
    summary: affected-set method gap recurs - re-signing and key-removing cards list consumers by hand and miss real ones (card 14 guard-step builders, card 20 recipe-entry test helpers and docs, card 13 prebuild path)
    fix: >-
      Re-derive the Edit list of every card that changes a field, signature, constant or key by searching the module, tests included, for the changed identifier and for the old literal, and listing the symbol that encloses each hit; confirm each glyph with lyx quarry resolve.
      Apply these three lists, which together pass lyx loom validate-plan --overlay with no finding beyond the baseline's.
      Card 14 - replace the Edit line for internal/gatecli#runGuardStep (no such function exists; the baseline overlay with {} already reports glyph-not-found for it) by internal/gatecli#guardStepRun -> func guardStepRun(cfg gateslot.Config, moduleDir string, tier int, guards map[string][]string, request testRequest) (tierRun, error), the function that today builds the guard step from cfg.GuardTier, and add internal/gatecli#guardPackages and internal/gatecli#execution (its guards field, run.go, carries the guard names the groups are built from).
      Replace every runGuardStep in the Intent by guardStepRun, and have the Intent say what type scope.guards and execution.guards carry once the groups are per tier.
      Card 20 - add to Edit internal/shedrecipe#commandRefWayForward (its text tells the writer to set switch), internal/shedrecipe#commandTestConfig and internal/shedrecipe#commandTestSettings (the helper config still holds the switch key that commandEntry now refuses as unknown, and conformance_test.go builds its Command row from commandTestConfig), internal/shedadapters#CommandProducer (its doc says the row runs when its switch is on), internal/battenshed#Config and internal/battenshed#ConfigSpec (their docs name the Command rows' switch keys).
      Add to the Intent that commandRefWayForward loses the word switch and that the helper config drops the key.
      Card 13 - add to Edit cmd/testtiming#prebuildLyx and cmd/testtiming#TestPrebuildLyx, and add to the Intent that the tier0 default reaches go test only, so run's tier label, its real-git hint and prebuildLyx still treat a run with no tags of its own as the offline tier and build no lyx.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/15-empty-tier-fails.md
    summary: card 15 lists internal/gatecli/doc.go under Extends although its Intent falsifies two existing sentences of that doc
    fix: >-
      In card 15 remove the Extends block and add internal/gatecli/doc.go as the last line of the Edit list.
      Add to the Intent that the Counters, record and log section is rewritten: the sentence saying the counts decide nothing about the outcome, and the sentence saying a tier that discovers no tests passes with zero counted as a package with only a TestMain does, both change to the D7 rule.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/17-shed-entry-gate-engine.md
    summary: card 17 lists internal/shedengine/doc.go under Extends although its Intent falsifies existing sentences of that doc
    fix: >-
      In card 17 remove the Extends block and add internal/shedengine/doc.go as the last line of the Edit list.
      Add to the Intent that the doc's existing sentences change to match: the Caller labels sentence that the next write appending a history entry hands pending_by to it (a skip write appends an entry and carries the label through), the pause_after sentence that it fires on a routed running outcome (a skipped row also fires it), the bounce-budget sentence that an episode ends at a Done entry (a skipped entry ends it as a Done does), and the Field ownership sentence listing the Shed-owned fields (entered_row joins it).
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/20-batten-switch-entry-gate.md
    summary: card 20 plans forced-answer assertions that existing battencli tests already cover, a reopen row that cannot differ from another row, and a switch row beside the unknown-key row
    fix: >-
      In card 20 keep the TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree assertion that Env.Setting answers batten.open_window true under --open-terminal and refuses without it; it is the one place the wired forced answer is pinned.
      Drop from the Intent the Env.Setting false-without and true-with assertions and the seed_ide assertion for stepCommandRows_FollowTheTasksLocalSwitch; that test keeps its marker assertions, which already prove the forced Open-Window run through the entry gate, with --open-terminal set in place of Forced, and the Seed-IDE marker tests already prove seed_ide.
      Drop the planned rewrite of the SwitchOffAndOpenTerminalForced row of TestBattenPreRun_ReopensTheWindowWithNoClientAttached and delete the row with the forced field: its fake Setting answers a plain bool, so a rewritten row equals PastOpenWindowWithNoClient.
      Drop the planned switch-key assertion of TestCommandEntry as a new case: update the expected text of its UnknownKey row to the new way forward, which a leftover switch key reaches by the same configRejectUnknown path, and delete the MissingSwitch, NonStringSwitch and ReferenceWithNoDot rows.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/22-tier-plan-docs.md, _lyx/plan/31-retag-internal-p-z.md
    summary: prose that names the gate verb's flags and the old tags is outside the searches the cards ran (docs/overview.md gate entry and fabricengine_test paragraph, crucible go:build prose)
    fix: >-
      Card 22, Intent, in the docs/overview.md sentence: the gate entry's synopsis (lyx gate test [-C dir] [--tier N]... [--all] [--guards]) also names --gate key, and the paragraph describing fabricengine_test files as go:build integration-tagged names tier1.
      Card 31, Intent, in the crucible sentence: the go:build tmux and go:build llm prose of crucible/README.md and crucible/review-prompt-template.md, which tells a writer which tag a live test carries, names tier2 and tier3, since the one-tag scan this card lands refuses a test file tagged tmux or llm alone.
      Neither change touches a target list, so no overlay is needed.
---

### [BLOCKING] Affected-set method gap recurs across cards 14, 20 and 13

**Location:** `_lyx/plan/14-guard-step-per-tier.md`, `_lyx/plan/20-batten-switch-entry-gate.md`, `_lyx/plan/13-testtiming-tiers.md`

**Issue:**
The cards that retire a field, re-sign a function or remove a recipe key name their consumers from a search the writer ran by eye, and each such card this round omits a real consumer.
This is one method defect, filed once as `design`; the instances below are what it produced on the tree at HEAD.

Card 14 (retires `GuardTier`, changes `Targets.Guards` to carry tiers).
The Edit list holds `internal/gatecli#runGuardStep`, which does not exist: `grep runGuardStep` over the module finds nothing, and `lyx loom validate-plan --overlay` with `{}` returns `glyph-not-found/14-guard-step-per-tier` for it.
The gate owns that glyph check; the finding is the consequence for the affected set.
The function that builds the guard step is `guardStepRun` (`internal/gatecli/tierrun.go:160`), which reads `cfg.GuardTier` at lines 161 and 167 and takes `guards map[string][]string`; it is not on the card.
Its helper `guardPackages` (`tierrun.go:177`) takes the same map, and `execution.guards` (`run.go:279`) feeds both; neither is on the card, and the Intent never says what type `scope.guards` and `execution.guards` take once the groups are per tier.
Card 14's own search claim (readers of `Targets.Guards` found "over the module's tests") is otherwise accurate: `TestTargetsFor`, `TestDerive_OverARepository`, `TestFindGuardTests` and `TestModuleGuardTests_OverAModule` are all listed, and `guard_tier` hits in `tiers.go`, `scope.go`, `scopes_integration_test.go` and `test_integration_test.go` fall inside listed symbols.

Card 20 (removes the Command row `switch` key and `CommandDeps.Forced`).
`commandTestConfig` (`internal/shedrecipe/entries_command_test.go:17`) returns `Config{"switch": ..., "command": ...}`; after the card `commandEntry` refuses `switch` as unknown, so every test and the conformance table row (`conformance_test.go:65`) that builds a Command row from this helper fails until the helper changes.
The helper and `commandTestSettings` (whose comment says "the switch on") are not on the card.
`commandRefWayForward` (`entries_command.go:19`) tells the writer to "set switch and command to module.key setting references", which the card makes false and leaves in place.
The `CommandProducer` type doc ("runs ... when the row's switch is on") and the `Config` and `ConfigSpec` docs of `internal/battenshed/config.go` (lines 24 and 49) also name the switch; `lyx quarry resolve` confirms those lines lie inside the glyphs `internal/battenshed#Config` and `internal/battenshed#ConfigSpec`.

Card 13 (default timing run becomes `-tags tier0`).
`run` (`cmd/testtiming/main.go:166-181`) branches on `tags != ""` three times: it appends `-tags`, labels the run "Tags: ..." instead of the offline tier, prints the "real local git" hint, and passes `tags` to `prebuildLyx`, which builds `cmd/lyx` for every non-empty tag set (`prebuild.go:24`).
A default run that simply carries `tier0` as its tag set would build `lyx` and export the prebuilt-binary variable, which no `tier0` test needs, and `TestPrebuildLyx` (rows with tags `""` and `"integration"` only) would stay green.
Neither `prebuildLyx` nor `TestPrebuildLyx` is on the card, and the Intent does not say where the `tier0` default is applied.
The same placement question reaches `redundancyTags`, which depends on `resolveTags` returning `""` to select `defaultRedundancyTags`; `TestRedundancyTags` (listed) would catch that one.

The fix key applies the three lists, with the pieces of Intent they need.
All three together passed `lyx loom validate-plan --overlay` with the F2, F3 fixes laid over, with no finding the baseline lacks.
The baseline's own `glyph-not-found` for `runGuardStep` disappears with the card 14 change.

### [MEDIUM] Card 15 extends a doc whose existing sentences its Intent falsifies

**Location:** `_lyx/plan/15-empty-tier-fails.md:18`

**Issue:**
`internal/gatecli/doc.go` sits under `**Extends:**`, which is for adding to a ref without changing anything already there.
The doc's Counters, record and log section says "the counts decide nothing about the outcome" (line 138) and "A tier that discovers no tests passes with zero counted, as a package with only a `TestMain` does" (line 143).
Card 15's Intent makes the per-package counts decide the outcome and makes a package whose untagged `TestMain` runs with zero tests fail the tier when its command lacks the tag.
Both sentences must be rewritten, so the ref belongs under `**Edit:**`.

### [MEDIUM] Card 17 extends a doc whose existing sentences its Intent falsifies

**Location:** `_lyx/plan/17-shed-entry-gate-engine.md:31`

**Issue:**
`internal/shedengine/doc.go` sits under `**Extends:**`, but the Intent changes what four existing statements say.
The Caller labels section says "The next write that appends a history entry hands pending_by to that entry and clears it; a write that appends none carries it through", and the Intent has a skip write append a `skipped` entry while carrying the label through.
The Field ownership section says "pause_after fires when the named producer returns a routed running outcome (done to OnDone, a bounced stuck to OnStuck)", and the Intent makes a skipped row fire it too.
The bounce-budget paragraph counts Stucks since the producer's "own most recent Done entry", and the Intent has `episodeStuckCount` treat a skipped entry as a Done.
The same section lists the Shed-owned fields, which gain `entered_row`.
Rewriting them is an `Edit`.

### [MEDIUM] Card 20 plans assertions existing battencli tests already cover

**Location:** `_lyx/plan/20-batten-switch-entry-gate.md`

**Issue:**
Three of the planned changes pin behavior another test already pins, or cannot differ from another row.

First, the forced answer of the wired `Env.Setting`.
`TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` (`wire_seams_test.go:229-263`) already asserts the wiring's flagged and unflagged behavior today through `Command.Forced`; the card moves that assertion to `Env.Setting`, which is right.
The card then has `stepCommandRows_FollowTheTasksLocalSwitch` (`lifecycle_integration_test.go`) assert the same answer a second time (true with `--open-terminal`) and add the false answer and an unchanged-`seed_ide` answer.
That test's marker assertions already prove both directions end to end, since an Open-Window marker exists only when the entry gate saw `true`, and the Seed-IDE marker tests in the same file cover `seed_ide`.
Round 2 proposed moving the deleted new test's two assertions into this step and the round 2 fixer applied it without dispute; this round re-judges them as redundant with those marker assertions, so the earlier fix is narrowed, not re-raised against a dispute.

Second, the `SwitchOffAndOpenTerminalForced` row of `TestBattenPreRun_ReopensTheWindowWithNoClientAttached` (`reopen_window_test.go:44`) drives `battenPreRun` over a fake `c.env.Setting` that answers one plain bool.
Rewritten to "answer true", it equals the `PastOpenWindowWithNoClient` row; the forced mechanism it used to pin lives in the wiring, covered above.

Third, the `switch`-key assertion of `TestCommandEntry`.
The table already holds an `UnknownKey` row asserting the way-forward text for a key the entry does not accept; a leftover `switch` key reaches the same `configRejectUnknown` branch with the same constant.
Updating that row's expected text pins the new wording; a further case pins nothing new.

### [LOW] Prose naming the gate flags and the old tags escapes the cards' searches

**Location:** `_lyx/plan/22-tier-plan-docs.md`, `_lyx/plan/31-retag-internal-p-z.md`

**Issue:**
`docs/overview.md:461`, the gate entry, gives the verb's synopsis as `lyx gate test [-C <dir>] [--tier N]... [--all] [--guards] [--wait] ...`; none of the cards 2, 7, 10 or 14 gives it `--gate <key>`, and card 22's sentence on this file covers only the paragraphs that name the retired `landing` and round gates.
`docs/overview.md:766` calls the `fabricengine_test` files "`//go:build integration`-tagged", which the retag makes `tier1`.
Card 31's crucible edit came from a search for `-tags` over non-Go files; the `//go:build tmux` and `//go:build llm` prose in `crucible/README.md:199` and `crucible/review-prompt-template.md` (lines 122, 129, 132, 203) was not found by it, and tells a writer to tag a live test with a spelling the one-tag scan card 31 lands then refuses.

## Rejected

- Card 14's `glyph-not-found` for `runGuardStep` as a finding of its own: the plan gate reports it; it appears in F1 only as evidence for the omitted `guardStepRun`, `guardPackages` and `execution`.
- Retag support files and old tag spellings (focus 2): every directory holding a `_test.go` file outside testdata (169) appears in a card 23 to 31 package glyph, so no package is missed.
  The support-only constraints in the tree are `!integration` (12 `TestMain` companions), `integration || tmux` (`loomcli/smoke_helpers_test.go`) and `tmux || llm` (`reedcli`, `creelcli` smoke helpers); the `retag-rules` decision respells each with its tier tag(s), and the transitional tier commands carry both spellings so each compiles where it did.
  The only non-test file under an old tag, `internal/testkit/tmuxkit/allow_tmux.go`, and the only testdata file under one, `internal/planglyph/testdata/glyphchain/shapes/tagged.go`, are covered by card 28 and the D13 follow-up in card 31.
  `mill-config.yaml`'s `done_gate` names `-tags integration` but belongs to the retired mill tooling and no gate reads it.
  The `-tags tmux` and `-tags llm` entries of `TestHubGenericStencils_NameNoGoTiers` add nothing beyond the `go test` entry beside them, so respelling them buys no coverage.
- Card 3's rendered-string sweep (focus 1): card 3 is done; the old `--tier ... --card` spelling remaining in `contracts/specs/loom-plan-spec.md` (lines 321, 331, 338) sits in the verify model section that card 22 rewrites, and the test fixtures with old spellings (`websterengine/template_test.go`, `landingshed/verifygate_test.go`) are stand-in strings that assert no rendering.
- Card 19's `Env.SettingKnown` and `Env.Resume` against the `intentionallyNil` allowlists of `loomcli` and `battencli`: both wires fill the fields, so neither allowlist gains an entry.
- Extends on cards 9, 10 and 19: the existing text of `gateslot/doc.go`, `gatecli/doc.go` (for the launch list and `report`), `docs/overview.md` and `shedbuild/doc.go` stays true, so these are additions.
