# Round 4 fixer report — plan

## Changes

- F1 (BLOCKING), fixed as proposed, with one difference in card 19:
  - Card 10: the Edit group gains `newTestEnv`, `TestResolveGateSpec_VerifyRequiresAbsoluteEnvRoots` and battencli's `intentionallyNil`. The Tests paragraph says `newTestEnv` tells a fixed `Round`, adds the `Round` nil row, and adds the battencli entry.
  - Card 12: the Edit group gains envkit's `nilLegal`, `TestRun_VerifyGate` and `gateScenario.restart`. The Tests paragraph adds the `nilLegal` entry and recasts `restart`: it commits a `gate.yaml` with a `final` stand-in, tells `Geom.GateConfig`, writes the `.lyx` exclude and puts `lyx` first on `PATH`. It also says the rows are not parallel.
  - Card 13: the Edit group gains `newGateFixture`, `TestWire_Real_PublishRejectedPushNamesRemoteTip` and battencli's `intentionallyNil`. The Intent says `newVerifyGate` sets the tier lookup on `verifyGate` from `Deps.GateTiers`. The Tests paragraph covers the fixture's lookup, the publish integration test's seams, the three battencli entries, and the `TestDarnTreeScope_ClassifiesAtEachCall` recast.
  - Card 15: the Delete group gains `guardFixture`, `guardSourceMarked` and `guardSourceMisplaced`. The Tests paragraph drops the guard rows of `TestBeginBatch_PromptFile` and `TestRecoverSpawnOrAttach`, and recasts the per-package step rows as the `--card` call.
  - Card 19: the `TestWebsterGeometry` sentence gains the `Tier0Parallel` literals, the machine `tier_parallel: {tier0: 0}` row and the override row. The proposal said to replace the whole sentence; I kept its lease-GOFLAGS drop, because the test still asserts GOFLAGS at `internal/hubgeom/webstergeom_test.go:156`. The proposal's first-`Limits()` clause moved to card 2 during the self-check (see below).
- F2 (BLOCKING), fixed as proposed:
  - Card 16's Intent makes `ErrConfigAbsent` at the loomcli and webstercli wiring loads no refusal: the `gate_tiers` marker renders `none`. Every other load error still refuses. `TestWire_LanguageSkills` and `TestWireHub_LanguageSkills` gain the no-`gate.yaml` row over a hub location, and the card says no wiring fixture needs seeding.
  - Card 18's Intent says `ErrConfigAbsent` leaves `RefuseRaw` empty at both loads. `TestValidatePlanCmd` gains the no-`gate.yaml` row, and the card says no validate fixture needs seeding.
- F3 (MEDIUM), fixed as proposed. Card 2's `decodeConfig` Edit line gains the re-sign arrow, `TemplateConfig` joins the Edit group, and the Intent says `TemplateConfig` passes false.

`lyx loom validate-plan` reports `ok: true` with informational findings only, both after the fixes and after the self-check edits.

## Self-check

- fix-consistency, MEDIUM: the first-`Limits()` assertion in hubgeom's `TestWebsterGeometry` breaks at card 2, which changes the no-board way forward, not at card 19. Fixed as proposed: the clause moved to card 2's Tests paragraph and `internal/hubgeom#TestWebsterGeometry` joined card 2's Edit group. Card 19 keeps its `Tier0Parallel` and `tier_parallel` rows.
- fix-claims: no findings.
- fix-proposal, MEDIUM: setting `deps.PublishCommand` and `deps.BaseCommand` to nil would panic, because no card makes a landing site check those readers for nil. Fixed with my own change: card 13 now stubs both as readers returning `""`, which takes the empty `publish` path. I did not make the fork's other option, a nil check in production, because `code-quality` rules out handling a state the wiring's own guarantees prevent.
- fix-proposal, LOW: `TestRun_VerifyGate` calls `t.Parallel()` at its top (`internal/websterengine/runlevel_git_test.go:68`), so a `PATH` change under it panics. Fixed as proposed: card 12 says the test drops its `t.Parallel()`.
- fix-affected-set, BLOCKING: card 12 fixes only `restart` among the Run tests that rely on the retired `## verify:` input. Deferred; see below.

## Disputed

None.

## Deferred

- Self-check fix-affected-set finding (BLOCKING), on card 12's Run fixtures. The premise holds. Today the webster verify gate passes with a warning when the plan has no `## verify:` section (`internal/websterengine/verifygate.go:384`). Every untagged Run test in `internal/websterengine/runlevel_test.go` relies on that, as `seedRunPlanDir`'s doc says, or appends `true` through `appendIntegrationVerify` (lines 2088, 2120, 2145, 2193, 2826 and 2856).
  From card 12 on, every Run test that reaches a `done` outcome renders and runs `lyx gate test --wait --tier <final> --all` through `verifytree.Verify` in `NewVerifyGate`'s own closure. Neither `RunDeps` nor `Geometry` lets a test fake that run. These are parallel tests over a `fakeGit`, so `lyx` would come from the operator's `PATH`, or be missing, and the gate's dirty and tree checks would hit a fake worktree.
  Adding `GateConfig` to `newRunFixtureOver` does not resolve it. The sound fix is a design choice between two options:
  - A told verify-run seam on `Geometry`. The fixture would fake it, while `runlevel_git_test.go` keeps pinning the real run. This would ripple into hubgeom's and standalonegeom's constructors, `shedfake.WebsterSeams`, envkit's `nilLegal`, battencli's `intentionallyNil`, and card 14's drop of the pool and timeout arguments.
  - Running a built `lyx` for the whole unit Run suite, through a package-level `PATH` step and a committed `gate.yaml` in every fixture.
  That choice crosses cards 12, 14 and 15 and the seam inventories, which is more than a self-check edit can settle safely. It needs the next round's design pass, which should also list `newRunFixtureOver`, `appendIntegrationVerify`, `seedRunPlanDir`'s doc and the callers above.
