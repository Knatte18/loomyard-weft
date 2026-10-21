---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/10-round-gate-tiers.md, _lyx/plan/12-webster-final-gate.md, _lyx/plan/13-landing-gates.md, _lyx/plan/15-webster-prompt-gates.md, _lyx/plan/19-non-tier-holders.md
    summary: The Edit groups still miss fixtures, reflection-driven seam inventories and tests bound to retired inputs that break under the cards' own new required seams
    fix: |-
      Apply these additions; each Edit/Delete line was checked together with the fixes of F3 through `lyx loom validate-plan --overlay` and adds no blocking finding.
      Card 10: add to Edit `internal/shedrecipe#newTestEnv`, `internal/shedrecipe#TestResolveGateSpec_VerifyRequiresAbsoluteEnvRoots` and `internal/battencli#intentionallyNil`.
      In its Tests paragraph say: `newTestEnv` tells a fixed `Round` (a fixed round list and a `GateTiers` over fixed gate lists), so `darnTestEnv`, `reworkTestEnv` and `TestDarnWriteEntry_HappyPath`, which resolve a `verify` gate through it, keep resolving; `TestResolveGateSpec_VerifyRequiresAbsoluteEnvRoots` gains a `Round` nil row beside `VerifyCommand`; battencli's `intentionallyNil` gains `Round` with the reason "loom-only seam, batten runs no verify gate".
      Card 12: add to Edit `internal/testkit/envkit#nilLegal`, `internal/websterengine_test#TestRun_VerifyGate` and `internal/websterengine_test#gateScenario.restart`.
      In its Tests paragraph say: `nilLegal` gains `WebsterDeps.Geom.GateConfig`, as it holds `WebsterDeps.Geom.Git` and `WebsterDeps.Geom.GateSlots`, because `FullEnv` takes its geometry from `shedfake.WebsterSeams()` and `TestFullEnv_FillsEverySeam` walks `Geometry`'s new func field, while `loomCLI.wire` fills it through `hubgeom.WebsterGeometry`; `TestRun_VerifyGate` drives Run's verify gate through `gateScenario.restart(t, verify)`, which today appends the command as the plan's `## verify:` section, so `restart` instead writes and commits `_lyx/config/gate.yaml` whose `final` tier runs `verify` as a shell stand-in, tells `Geom.GateConfig` a reader over it, writes `lyxdirs.DotLyxDirName` plus a slash into the repo's `.git/info/exclude` and puts the tree's `lyx` first on `PATH` through `lyxbin.PutFirstOnPath`; the scenario rows are not parallel, naming that process-global state (card 15's `newRunFixtureOver` config declares only `tier0` and a `round` list, so `restart` must override it).
      Card 13: add to Edit `internal/landingshed#newGateFixture`, `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip` and `internal/battencli#intentionallyNil`.
      In its Intent say the tier lookup is a field of `verifyGate`, set by `newVerifyGate` from `Deps.GateTiers`; in its Tests paragraph say: `newGateFixture` hand-builds `verifyGate` and is the gate that `newMergeInFixture` and `newPublishVerifyFixture` install, so it sets the lookup over fixed gate lists (`Deps.GateTiers` set by `newPublishVerifyFixture` never reaches a hand-built gate), which `TestMergeIn_PlanVerifyFailureRecord` and `TestPublishVerify_FailureRecord` need for their logs without a summary line; `TestWire_Real_PublishRejectedPushNamesRemoteTip` runs over a tierless `hubforge` hub, where `TierList` refuses a `PublishCommand` read for lack of `tier0`, so it sets `deps.PublishCommand = nil` and `deps.BaseCommand = nil` beside `deps.VerifyCommand = nil`; battencli's `intentionallyNil` gains `Landing.PublishCommand`, `Landing.BaseCommand` and `Landing.GateTiers` with the reason "loom-only seam, batten lands no branch"; `TestDarnTreeScope_ClassifiesAtEachCall` seeds the hub's `gate.yaml` with a `tier0` and a `landing` list, and its non-scoped `assertLanding` rows, which expect darn's retired `verify` value `run all`, assert the rendered landing call `lyx gate test --wait --tier 0 --all`.
      Card 15: add to Delete `internal/websterengine_test#guardFixture`, `internal/websterengine_test#guardSourceMarked` and `internal/websterengine_test#guardSourceMisplaced`.
      In its Tests paragraph say: `TestBeginBatch_PromptFile` drops its guard rows (the marked-test row, the two misplaced-marker rows and the over-the-command-limit row that names `impactset.MaxCommandLength`) and recasts its remaining rows that expect the per-package `lyx gate test ./internal/alpha` step as the rendered `--card` call; `TestRecoverSpawnOrAttach` drops its row "a marked guard test renders the same guard step the begin prompt carries".
      Card 19: its `TestWebsterGeometry` sentence says only that the test drops its lease GOFLAGS assertion; replace it with: the `Limits` literals take `Tier0Parallel` in place of `GoParallel`; the machine `go_parallel: 0` row, which asserts a load error, now loads because the key is retired, so it becomes a machine `tier_parallel: {tier0: 0}` row over a seeded `tier0`, refused naming both files; the machine override row seeds `tier_parallel.tier0`; and the first `Limits()` assertion, made before the test creates the board directory so `fabricengine.ConfigDirs` tells no board and card 2's no-board way forward applies, expects `lyx config gate` and the `_lyx` clause in place of `lyx fabric reconcile`.
  - id: F2
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/16-retire-verify-tiers.md, _lyx/plan/18-plan-gate-verify-checks.md
    summary: Cards 16 and 18 load gate.yaml strictly at wiring and refuse an absent file, which breaks every hub or anchor fixture without one and contradicts card 17 and wire's tolerant-hub contract
    fix: |-
      In card 16's Intent, where loomcli's wiring and webstercli's hub wiring load `gate.yaml` through `gateslot.LoadConfig`, state that `errors.Is(err, gateslot.ErrConfigAbsent)` is no refusal there: the `gate_tiers` marker renders `none`, as card 16 already says for a standalone worktree, and every other load error still refuses naming `lyx config gate`.
      In card 18's Intent, state the same for `validatePlanCmd`'s load and for the wiring that fills `Env.RefuseRaw`: `ErrConfigAbsent` leaves `RefuseRaw` empty, as card 17's `Claude.Prepare` does, and every other load error refuses the verb.
      In card 16's Tests paragraph, add to `TestWire_LanguageSkills` and `TestWireHub_LanguageSkills` the row "no gate.yaml, wire succeeds and the marker renders none"; in card 18's, add to `TestValidatePlanCmd` the row "no gate.yaml, the verb runs and `verify-raw-command` reports nothing".
      No fixture needs seeding then: loomcli's `hubLocation` and its callers, loomcli's `hubLocationAt` and `glyphPlanFixture` validate fixtures, and webstercli's `hubLocation` with `TestWire_ModeHubSelectsHubMode` and its siblings keep loading as they do today.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/02-gate-tier-config.md
    summary: decodeConfig gains a parameter without the re-sign arrow and its production caller TemplateConfig is not in the Edit group
    fix: |-
      In card 2's Edit group replace the line
      - `internal/gateslot#decodeConfig`
      with
      - `internal/gateslot#decodeConfig` -> `func decodeConfig(data []byte, machineLayerApplies bool) (Config, error)`
      - `internal/gateslot#TemplateConfig`
      and in the Intent paragraph on the dropped `tier_parallel` entry add that `TemplateConfig` passes false, since the embedded template has no machine layer.
      This overlay passes `lyx loom validate-plan --overlay` with no new blocking finding.
---

### [BLOCKING] The Edit groups still miss fixtures, seam inventories and tests bound to retired inputs

**Location:** `_lyx/plan/10-round-gate-tiers.md`, `_lyx/plan/12-webster-final-gate.md`, `_lyx/plan/13-landing-gates.md`, `_lyx/plan/15-webster-prompt-gates.md`, `_lyx/plan/19-non-tier-holders.md`

**Issue:** Focus questions 1 and 2 ask for every fixture and caller that breaks under a card's changed constructor, required reader, deps struct or skip rule.
The plan finds callers by following each symbol through production code and the tests that name it.
It misses three classes that never name the symbol, and the same method gap repeats on four cards, so it is filed once here.
Round 3's F1 filed the same gap for cards 2, 13 and 15; its fixes hold, and every instance below is a different one, confirmed against the repo.
None of them re-raises the card 15 point that the round-3 fixer disputed (the wider set of tests built on the begin, recover and run fixtures), which this review leaves settled.
- Reflection-driven seam inventories.
  `envkit.NilSeams` walks `shedrecipe.Env` and every nested struct for nil func fields.
  `battencli`'s `TestWire_EverySeamFilled` (`internal/battencli/wire_seams_test.go`) requires each nil seam in `intentionallyNil`, so card 10's `Env.Round` and card 13's `Landing.PublishCommand`, `Landing.BaseCommand` and `Landing.GateTiers`, which batten leaves nil, fail it.
  `envkit.TestFullEnv_FillsEverySeam` fails on card 12's `Geometry.GateConfig`, because `FullEnv` takes its geometry from `shedfake.WebsterSeams()`, which sets only `Index`.
  Card 13 says `GateTiers` stays out of `nilLegal`, so the battencli map is the route for it.
- Hand-built fixtures that bypass the production constructor.
  Card 10 makes `Round` required by the `verify` gate entry, but `newTestEnv` (`internal/shedrecipe/fixture_test.go`) sets only `VerifyCommand`, so `TestResolveGateSpec_VerifyRequiresAbsoluteEnvRoots` (its success half) and `TestDarnWriteEntry_HappyPath` stop resolving.
  Card 13's tier lookup on `verifyGate` has no source in `newGateFixture` (`internal/landingshed/verifygate_test.go`), which both `newMergeInFixture` and `newPublishVerifyFixture` install by hand (`&Publish{deps: deps, gate: fx.gate.gate}`), so `Deps.GateTiers` set in `newPublishVerifyFixture` never reaches the gate.
  `TestMergeIn_PlanVerifyFailureRecord` (log `/verify/verify-1.log`) and `TestPublishVerify_FailureRecord` (log text with no summary line) take exactly the fallback path the card says would panic on a nil lookup.
  `TestWire_Real_PublishRejectedPushNamesRemoteTip` builds a tierless `hubforge` hub and calls `Publish`, which now reads `PublishCommand`, and `TierList` refuses a hub with no `tier0` even for an empty `publish` list.
  `TestDarnTreeScope_ClassifiesAtEachCall` asserts darn's retired `verify` value `run all` in its non-scoped rows over the same kind of hub.
- Tests bound to a retired input.
  `TestRun_VerifyGate` and `gateScenario.restart` (`internal/websterengine/runlevel_git_test.go`) feed the gate its command as the plan's `## verify:` section, which card 12 stops reading, and `newRunFixtureOver` (card 15) declares no `final` list; no card lists either.
  Card 15 deletes the Go-side guard scan, but `TestBeginBatch_PromptFile`'s guard rows (including the one that names `impactset.MaxCommandLength`) and `TestRecoverSpawnOrAttach`'s "a marked guard test" row still assert the old `lyx gate test --tags integration ./internal/g -- -run` step, and the helpers `guardFixture`, `guardSourceMarked` and `guardSourceMisplaced` would be left unused.
  Card 19's `TestWebsterGeometry` sentence misses three rows: the `Limits` literals that name `GoParallel`, the machine `go_parallel: 0` row that expects a load error the retirement removes, and the first `Limits()` assertion, made before the board directory exists, which expects `run "lyx fabric reconcile"` where card 2's no-board way forward now names `lyx config gate`.

Focus question 1's second half has a narrow answer: no fixture builds a literal zero `gateslot.Config`; the ones that meet a refusing reader are the tierless hubs and the missing-file hubs named above and in F2.

### [BLOCKING] Cards 16 and 18 refuse an absent gate.yaml at wiring

**Location:** `_lyx/plan/16-retire-verify-tiers.md`, `_lyx/plan/18-plan-gate-verify-checks.md`

**Issue:** `gateslot.LoadConfig` is strict: an absent hub file is `ErrConfigAbsent`, and with no board told the worktree file is read, so a missing file is a load failure.
Card 16 says a failure at the load in `loomCLI.wire` and `websterCLI.wireHub` refuses the verb, with only a standalone worktree rendering `none`; card 18 says the same of `validatePlanCmd` and the wiring that fills `Env.RefuseRaw`.
`languageconfig.Load`, which these loads sit beside, degrades to the template on an absent file, and card 17 treats `ErrConfigAbsent` as an empty list.
`wireHub`'s documented contract (`internal/webstercli/wiring_test.go` header) is that every module load degrades on an absent `_lyx/`, and `TestWire_ModeHubSelectsHubMode` asserts a healthy but unwired hub location "must never be refused".
loomcli's `hubLocation` (used across the wiring, darn and landing-deps tests) seeds `loom.yaml` and `landing.yaml` but no `gate.yaml`, and the validate-plan fixtures (`glyphPlanFixture`, `hubLocationAt`) have none either.
No card lists those fixtures, so the plan as written fails them all; the fix removes the need by making an absent file a non-refusal, which also keeps `lyx loom validate-plan` usable in a standalone repo with no `gate.yaml`.

### [MEDIUM] decodeConfig gains a parameter without the arrow, and TemplateConfig is not an Edit target

**Location:** `_lyx/plan/02-gate-tier-config.md`

**Issue:** The Intent says `LoadConfig` tells `decodeConfig` whether `configengine.LayerFile` reports the machine layer applies, so `decodeConfig(data []byte)` gains a parameter, but its Edit line has no re-sign arrow.
`internal/gateslot/config.go` has a second production caller, `TemplateConfig`, which calls `decodeConfig([]byte(ConfigTemplate()))` and would stop compiling; it is in no card's Edit group.

## Rejected

- Focus question 2, sources of the tier list: `checkedPublishFailure` and `publishFailureSteps` take the lookup from `NewVerifyGate`'s `round.GateTiers` (card 10), `PublishFailureNote` from the closure in `loomCLI.wire`, which builds it from `gateslot.LoadConfig` at call time, and `verifyGate.recordFailure` from `Deps.GateTiers` (card 13); every production caller of those re-signed functions, and of `verifytree.Verify`, `PlanTable`, `seedDarnHubConfig` and the webster render functions, is in its card's Edit group. The one gap is the hand-built gate fixtures, filed in F1.
- Focus question 3: card 10 (`TestVerifyGate_RoundScenario`) and card 12 (`newVerifyGateRepo`, `newVerifyFixture`, `TestGateParity_VerifyGate` through the same fixture) state that the seeded `gate.yaml` is committed and that `lyxdirs.DotLyxDirName` plus a slash goes into `.git/info/exclude`. The other built-`lyx` tests run in a hub (run records go to the board, not the worktree) or in a standalone worktree that makes no later diff or clean-tree call, so they need neither statement.
- Regression check of the earlier rounds' fixes: card 2 still lists `TestEditOne`, `TestDispatchSet` and the two-step drop warning; card 10 still carries `RoundGate.GateTiers` and the `tiers` argument; card 13 still carries `Deps.GateTiers` and the arrowed `seedDarnHubConfig`; card 15 still seeds the begin, recover, run and verbs fixtures; cards 10 and 12 still state the commit and the exclude. None regressed, and the round-1 and round-2 fixer reports defer nothing.
- Focus question 4: the dropped `tier_parallel` warning names plain `lyx config gate` for the hub file and `lyx config gate --layer machine` for the machine file, and the standalone case names plain `lyx config gate` alone; the other refusals name keys that sit in the hub file only (`tiers`, `gates`, `scoper`, `guard_tier`, `refuse_raw`, darn's `tree_scope_verify`), where `lyx config gate` or `lyx config darn` edits the one layer. No refusal names a step that cannot reach the layer holding the entry.
