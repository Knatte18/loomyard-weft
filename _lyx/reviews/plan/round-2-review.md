---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/03-gate-call-renders-key.md:3
    summary: The cards list the readers of what they change from constructor and fixture searches, so tests that assert a changed rendered call, field type or config key stay unlisted on cards 3, 4, 5, 14 and 20
    fix: For every card that changes a field, key, signature, rendered string or value type, build the reader list from a search of the module's tests and docs for the changed name and for the old literal, and list each hit on that card or an earlier one. Apply it to these cards. Card 3 Edit group, add internal/websterengine#TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach, internal/websterengine#TestRenameBoundary_BatchesCheckAndGateTheirOwnTree, internal/websterengine_test#TestBeginBatch_PromptFile, internal/websterengine_test#TestRecoverSpawnOrAttach, internal/webstercli#TestRebaselineCmd_EditedCardOfFailedBatchThenRecover, internal/loomcli#TestWire_DarnRecipe, internal/loomcli#TestWire_DefaultConfig, internal/loomcli#wantDarnLandingCall, internal/loomcli#TestDarnTreeScope_ClassifiesAtEachCall and internal/loomshed#roundGateFixture.round, and say in the Intent that they assert the --gate spelling (--gate card --card <id>, --gate batch with one --card per card, --gate finalize for the landing call) and that the fixture's Render ignores scope words as gateListSource's does until card 4 deletes it. Card 4 Edit group, add internal/battencli#intentionallyNil and say its Round row goes with the field. Card 5 Edit group, add internal/landingshed#TestVerifyGate_RealVerifytreeScenario as a test that sets deps.VerifyCommand to a keyed closure. Card 14 Edit group, add internal/impactset#TestDerive_OverARepository and say its two Guards assertions compare GuardTest values with their tier. Card 20 Edit group, add internal/shedbuild#engineMinimalConfig, whose Command entry drops the switch key, and internal/battenshed/doc.go and docs/overview.md, whose Command rows paragraphs describe the entry gate in place of the switch. Overlay-validated with every other fix of this review.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/00-overview.md:59
    summary: The retag rules and the D13 follow-up entry leave three kinds of old-spelling tag consumers that break once the follow-up drops the old tags from the hub commands
    fix: In the overview's retag-rules decision and card 23's rule paragraph, let a TestMain companion constrained !integration take !tier1, which holds under the hub's tier1 command in both spellings. In card 28 retag the planglyph fixture glyphchain/shapes/tagged.go to tier1 and give internal/testkit/tmuxkit's non-test allow_tmux.go the constraint tier2 || tier3 || tmux || llm with a comment that names the tier tags, keeping the tmuxkit package glyph since a file glyph beside it fails redundant-file-target. In card 31's follow-up entry text add the tmux and llm spellings of allow_tmux.go to the list of old spellings to remove. Overlay-validated with every other fix of this review.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/20-batten-switch-entry-gate.md:24
    summary: Card 20 creates TestWireSetting_OpenTerminalForcesOpenWindow for a forced answer that stepCommandRows_FollowTheTasksLocalSwitch and TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree already pin over the same wiring
    fix: Delete the Create group and its Intent paragraph from card 20 and extend the already listed internal/battencli#stepCommandRows_FollowTheTasksLocalSwitch with the two assertions the new test adds, that Env.Setting for batten.open_window answers false without --open-terminal and true with it and that seed_ide is unchanged. Overlay-validated with every other fix of this review.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/21-darn-describe-entry-gate.md:19
    summary: Card 21 lists internal/loomrecipe/doc.go under Extends although its darn paragraph names the edge rewrite the card removes
    fix: Move internal/loomrecipe/doc.go from card 21's Extends group to its Edit group and drop the Extends group, since the doc line saying a budget of 0 routes Darn-Write and Finalize straight to PR-Review is rewritten. Overlay-validated with every other fix of this review.
---

### [BLOCKING] The affected-set method keeps missing readers

**Location:** `_lyx/plan/03-gate-call-renders-key.md:3`, `_lyx/plan/04-verify-gate-site-key.md:3`, `_lyx/plan/05-landing-row-keys.md:3`, `_lyx/plan/14-guard-step-per-tier.md:8`, `_lyx/plan/20-batten-switch-entry-gate.md:3`

**Issue:** Round 1 filed one `design` finding for readers that retirement cards enumerated by hand, and its fix told each card to search for the changed field, key or symbol.
The cards now cite searches such as `Deps{` and `verifyGate{` over the landing tests, but the readers below are found by the changed name or by the old output text, and five cards still miss them.
This is the same gap, so it is filed once as a method finding with the per-card misses as its evidence.

- Card 3 renders `lyx gate test --wait --gate <key>` and keeps `gateListSource` rendering `GateCall(gate)`, and lists `TestCardGateCommand` and `TestRenderBatchGate` as the tests that assert the new spelling.
  These tests render through the changed functions and assert the `--tier` form:
  - `internal/websterengine/cardgate_test.go:73-77`, `TestRenderCardGates_OneBulletPerCardWithAStepSubBulletEach`, and `:141-142`, `TestRenameBoundary_BatchesCheckAndGateTheirOwnTree`.
  - `internal/websterengine/beginbatch_test.go:561-562`, `TestBeginBatch_PromptFile`, and `internal/websterengine/recoverbatch_test.go:1714-1715`, `TestRecoverSpawnOrAttach`; card 1 lists both only for their `Gates` seeds.
  - `internal/loomcli/wiring_darn_test.go:29,104,112,141`, `TestWire_DarnRecipe`, and `internal/loomcli/wiring_test.go:543`, `TestWire_DefaultConfig`, through the constant `wantDarnLandingCall` (`--tier 0 --tier 1 --all`); card 4 lists the two tests, but only at its later re-sign.

  All of these are untagged, so they run in card 3's card gate, whose packages are `websterengine` and `loomcli`, and fail there.
  Three integration-tagged readers fail at the batch gate unless card 4 shares the batch: `internal/webstercli/verbs_test.go:987` (`TestRebaselineCmd_EditedCardOfFailedBatchThenRecover`, listed on no card), `internal/loomcli/darntreescope_integration_test.go:39,122` (`landingCall`), and `internal/loomshed/verifygate_integration_test.go:222`, where `roundGateFixture.round` passes scope words to `GateCall`, which now reads them as card ids and renders `--card --base <sha>`.
- Card 4 deletes `Env.Round`, but `internal/battencli/wire_seams_test.go:37` holds `"Round"` in `intentionallyNil`.
  `TestWire_EverySeamFilled` fails for a listed path that `envkit.NilSeams` no longer reports, and the failure first appears in card 20's card gate, which edits battencli, far from its cause.
- Card 5 re-signs `Deps.VerifyCommand`, but `internal/landingshed/verifygate_integration_test.go:60`, inside `TestVerifyGate_RealVerifytreeScenario`, still assigns a zero-argument closure.
  The file is `integration`-tagged, so the build fails under `go vet -tags integration`, the first step of tier1 in the batch gate.
- Card 14 makes `Targets.Guards` carry `GuardTest` values, but `internal/impactset/derive_integration_test.go:102` and `:162`, in `TestDerive_OverARepository`, compare `got.Guards` with `map[string][]string{"g": {"TestGuard"}}`.
  `reflect.DeepEqual` compiles against the new type and returns false, so the tier1 run fails.
  Card 11 lists the test for its tag row, but card 14 does not.
- Card 20 makes `commandEntry` refuse a `switch` key, but `internal/shedbuild/build_engines_test.go:49` builds the `Command` engine with `"switch": "batten.open_window"` in `engineMinimalConfig`, so `TestBuild_EveryRegisteredEngineBuilds/Command` fails.
  `shedbuild` is not one of card 20's packages, so the first gate to run it is the whole gate at the end.
  `internal/battenshed/doc.go:73-79` and `docs/overview.md:620,636` also still say each Command row runs only when its `batten.yaml` switch is on and that a resume reopens the window through the same switch.

The cards' `ImpactSummary` lines name the packages they touch, not the readers of the changed output, so the plan gate's `caller-uncovered` check, which sees only re-signed members, cannot catch any of these.

### [MEDIUM] The retag rules and the follow-up entry miss the support files that name old tag spellings

**Location:** `_lyx/plan/00-overview.md:59`

**Issue:** The retag-rules decision leaves a support-only file's constraint as it is, and the D13 follow-up entry removes the old spellings from the hub commands, the package loads, `cmd/testtiming` and three guards.
Three consumers of the old spellings are in neither list, and each breaks once the hub's tier commands drop `integration`, `tmux` and `llm`:

- Twelve `testmain_test.go` files are constrained `!integration` and sit beside a `testmain_integration_test.go` that declares its own `TestMain` (`internal/gitkit/testmain_test.go:1`).
  After the retag the sibling is `tier1`; with `-tags tier1` alone `!integration` is true and both `TestMain` definitions compile together.
- `internal/testkit/tmuxkit/allow_tmux.go:1` is a non-test file constrained `tmux || llm` that sets `tmuxAllowed`.
  Without those tags `checkSockets` reports every tmux test's server in tier2 and tier3 binaries.
- `internal/planglyph/testdata/glyphchain/shapes/tagged.go:1` is constrained `integration`, and `glyphchain_test.go:211` resolves its member through the type load card 11 widens; the follow-up drops `integration` from that load.

`!tier1` holds under every spelling of the tier1 command and is false only there, so the companion keeps yielding to the `tier1` `TestMain`.
Card 23's rule paragraph says a support file keeps its constraint, and card 28 leaves the planglyph testdata alone.

### [MEDIUM] Card 20 duplicates the forced-answer coverage in a new integration test

**Location:** `_lyx/plan/20-batten-switch-entry-gate.md:24`

**Issue:** `TestWireSetting_OpenTerminalForcesOpenWindow` builds batten's `Env.Setting` over a hubforge task worktree with `open_window: false` and asserts the forced answer.
`stepCommandRows_FollowTheTasksLocalSwitch` (`internal/battencli/lifecycle_integration_test.go:1254`) already runs over a hubforge fixture with `open_window: false` and asserts that `--open-terminal` forces Open-Window on, and card 20 itself rewrites it onto the flag.
`TestWire_MarkerAndReportSeamsRefuseAnAbsentTaskWorktree` asserts the wire-level forced answer (`wire_seams_test.go:259-264`), and the `SwitchOffAndOpenTerminalForced` row of `TestBattenPreRun_ReopensTheWindowWithNoClientAttached` asserts it at `reopenWindow`.
The only behavior the new test adds is the unforced `false` answer and the unchanged `seed_ide`, which two assertions in the existing step cover.

### [LOW] Card 21 types a rewritten doc as an extension

**Location:** `_lyx/plan/21-darn-describe-entry-gate.md:19`

**Issue:** `internal/loomrecipe/doc.go:23` says a budget of 0 routes `Darn-Write`'s Done and `Finalize`'s reroute straight to PR-Review so a run reaches neither the perch nor `Describe`.
Card 21 removes that rewrite and states the new behavior in the same doc, so the existing line is rewritten and the ref belongs under Edit, not Extends.

## Rejected

- Headroom order of the four cards that grow `PATTERN.md`: the byte deltas of cards 3, 7, 19 and 31, measured from the current lines and the verbatim replacement lines, are 0, -7, -6 and -8, so each card's overview is no longer than the previous commit's and `pattern.Headroom` reports nothing at any card gate.
  The nineteen shortenings and the four additions also total net negative, which the plan gate's projection admits.
- Typing of the retag cards: an `Edit` over package self glyphs with an `ImpactSummary` is admitted, since `edit-test-file` reads only file self glyphs and `redundant-package-target` fires only beside a member of the same package; `Custom` would be the last resort the rubric forbids, and `Prosa` is for docs.
  The one-tag scan sits on the last retag card as D5 places it, and the retag cards cover every directory that holds a test file, testdata excluded.
- Re-sign arrows on cards 4, 5, 6 and 14: every member whose Intent changes its parameters or results carries the arrow, and the test callers of `NewVerifyGate`, `newGateFixture`, `withTreeScope`, `ModuleGuardTests` and `FindGuardTests` are listed, bar the readers named in F1.
- Entry-gate refusals: the chain-too-long stop, the evaluation error and the build refusals each name a way forward that exists in the tree (`lyx shed goto` admits a failed run's rows, `lyx config <module> --resolved`, `--layer local --set` and the `lyx loom resume` and `lyx batten run <slug>` resume verbs), and each has a named test row that reaches it through `Step` or `Build`.
- Retirement readers of `Config.Gates`, `guard_tier` and `tree_scope_verify`: the remaining readers I found by search (configcli editor refusal and its rows, `docs/overview.md`, darnengine constants and template, the gatecli test seeds, `scopesConfig`, `scenarioConfig`) are on cards 1, 6 and 14.
- Round 1's fixes hold: the `PATTERN.md` order, `Edit` retag typing, the re-sign arrows on `darnTreeScope` and `guardRunOf`, `TestDarnRouting` asserting no gate, the local-layer and resume way forwards, the `--apply` reconcile text, the `covers: []` Verify pattern, the follow-up type label and the dropped empty-key default all read as the fixer report says.
