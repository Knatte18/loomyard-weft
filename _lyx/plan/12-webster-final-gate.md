# Card 12 — webster-final-gate

**Create:**
- `plan:internal/webstercli#TestVerifyVerb_GateConfigRefusalIsTheError` -> `func TestVerifyVerb_GateConfigRefusalIsTheError(t *testing.T)`

**Edit:**
- `internal/websterengine#Geometry`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine#newVerifyGate`
- `internal/websterengine#TestVerifyGate_PassesWithoutVerifying`
- `internal/websterengine#newVerifyGateRepo`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI.verifyCmd`
- `internal/webstercli#websterCLI.runVerify`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/webstercli#newVerifyFixture`
- `internal/webstercli#TestVerifyVerb_PassRecordsTheTree`
- `internal/webstercli#TestVerifyVerb_SecondCallOnTheSameTreeSkips`
- `internal/webstercli#TestVerifyVerb_FailureExitsOneWithFindings`
- `internal/webstercli#TestVerifyVerb_DirtyTreeExitsOneNamingPaths`
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom#TestWebsterGeometry`
- `internal/standalonegeom#WebsterGeometry`
- `internal/standalonegeom#TestWebsterGeometry`
- `internal/testkit/envkit#nilLegal`
- `internal/websterengine_test#TestRun_VerifyGate`
- `internal/websterengine_test#gateScenario.restart`
- `contracts/specs/webster-spec.md`

**Delete:**
- `internal/webstercli#TestVerifyVerb_NoVerifySectionExitsZeroWithWarning`

**Uses:**
- `plan:internal/testkit/lyxbin#PutFirstOnPath`

**Intent:**
Webster's plan-level verify gate and `lyx webster verify` run the hub's `final` gate over the whole tree (D8), never the plan's `## verify:` section (D9).

`Geometry` gains `GateConfig func() (gateslot.Config, error)`, the told reader of the hub's `gate.yaml`, read at each call.
The two geometry constructors set it, so every builder of a webster geometry, `webstercli`'s wiring and loom's (`internal/loomcli`'s wiring and subjects) alike, carries it: `hubgeom.WebsterGeometry` tells `gateslot.LoadConfig` over the hub's config dirs, and `standalonegeom.WebsterGeometry` the same loader over the worktree's own directory, so a standalone run reads the worktree's `gate.yaml` as the gate verb does outside every hub, and one with none is refused naming `lyx config gate` (card 2).
The verify gate's command seam, and `runVerify`, render `GateConfig().GateCall("final", "--all")`, `lyx gate test --wait --tier <final list> --all`, and run it through `verifytree.Verify` under the same site labels; the rendered command is the verified-tree record's key, so Merge-In skips on a tree this gate passed when `final` and `landing` hold the same list (card 13).
Both sites still pass `verifytree.Verify` today's pool and timeout arguments, which card 14 drops.
A config load error or a `TierList` refusal is the gate's returned error at that arrival, naming `lyx config gate`, never a pass; the plan-has-no-verify warning path goes, since the gate no longer reads the plan's section.
The gate keeps its outcome check, dirty check, pre-fix head, rerun and flake logic; its rerun re-runs the same call.
`runVerify`'s help text names the `final` gate in place of the plan's section, and its no-section `verified: false` answer goes.

`doc.go`'s verify-gate paragraphs and `contracts/specs/webster-spec.md`'s verify-gate paragraph say the gate runs the `final` gate's rendered call.

Tests: `TestVerifyGate_PassesWithoutVerifying` keeps its outcome rows and swaps the no-section row for a gate-config refusal returned as the gate's error.
`newVerifyGateRepo` tells a `GateConfig` reader over a seeded `gate.yaml` whose `final` tier runs a shell stand-in.
The seeded `gate.yaml` is written to the scratch repo's own `_lyx/config/gate.yaml`, the file the built `lyx` reads there, and committed, and `newVerifyFixture` and `newVerifyGateRepo` write `lyxdirs.DotLyxDirName` followed by a slash into the repo's `.git/info/exclude`, since a standalone `lyx gate test` writes its run record and log under the worktree's `.lyx` gate directory and `verifytree` refuses an untracked path; `TestVerifyVerb_SecondCallOnTheSameTreeSkips` rests on that exclude, and the told `GateConfig` reader reads the same committed file.
The integration rows put the `lyx` built from the tree first on `PATH` through `lyxbin.PutFirstOnPath`, so they run the rendered call through this tree's `lyx gate test`; those tests are not parallel, naming that process-global state.
`TestGateParity_VerifyGate` builds both gates over the same seeded config, with the same `PATH` step.
The verb's own tests in `verify_integration_test.go` follow: `newVerifyFixture` tells a `GateConfig` reader over a seeded `gate.yaml` whose `final` tier runs the fixture's command as a shell stand-in, and the plan it writes no longer carries a `## verify:` section.
Its rows drive `runVerify` through the `lyx` built from the tree, first on `PATH` through `lyxbin.PutFirstOnPath`, so they are not parallel, naming that process-global state.
`TestVerifyVerb_NoVerifySectionExitsZeroWithWarning` asserts the answer this card removes, so it goes, and the new `TestVerifyVerb_GateConfigRefusalIsTheError` covers an unloadable config and a `final` list naming an undeclared tier, each returned as the verb's error naming `lyx config gate`.
`TestWebsterGeometry` of both geometry packages asserts the constructor sets `GateConfig`, reading a seeded file and, for the standalone one, refusing a worktree with none naming `lyx config gate`.
`nilLegal` gains `WebsterDeps.Geom.GateConfig`, as it holds `WebsterDeps.Geom.Git` and `WebsterDeps.Geom.GateSlots`, because `FullEnv` takes its geometry from `shedfake.WebsterSeams()` and `TestFullEnv_FillsEverySeam` walks `Geometry`'s new func field, while `loomCLI.wire` fills it through `hubgeom.WebsterGeometry`.
`TestRun_VerifyGate` drives Run's verify gate through `gateScenario.restart(t, verify)`, which today appends the command as the plan's `## verify:` section; `restart` instead writes and commits `_lyx/config/gate.yaml` whose `final` tier runs `verify` as a shell stand-in, tells `Geom.GateConfig` a reader over it, writes `lyxdirs.DotLyxDirName` followed by a slash into the repo's `.git/info/exclude` and puts the tree's `lyx` first on `PATH` through `lyxbin.PutFirstOnPath`.
`TestRun_VerifyGate` drops its `t.Parallel()` and its scenario rows are not parallel, naming that process-global state, and `restart` overrides the config of card 15's `newRunFixtureOver`, which declares only `tier0` and a `round` list.

**ImpactSummary:** the webster gate and its self-check verb read `gate.yaml` instead of the plan; both geometry constructors tell the new reader, so every webster geometry carries it, and no state or record format changes.
