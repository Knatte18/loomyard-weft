# Card 13 — webster-final-gate

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
- `internal/hubgeom#WebsterGeometry`
- `internal/hubgeom#TestWebsterGeometry`
- `internal/standalonegeom#WebsterGeometry`
- `internal/standalonegeom#TestWebsterGeometry`
- `contracts/specs/webster-spec.md`

**Uses:**
- `plan:internal/testkit/lyxbin#PutFirstOnPath`

**Intent:**
Webster's plan-level verify gate and `lyx webster verify` run the hub's `final` gate over the whole tree (D8), never the plan's `## verify:` section (D9).

`Geometry` gains `GateConfig func() (gateslot.Config, error)`, the told reader of the hub's `gate.yaml`, read at each call.
The two geometry constructors set it, so every builder of a webster geometry, `webstercli`'s wiring and loom's (`internal/loomcli`'s wiring and subjects) alike, carries it: `hubgeom.WebsterGeometry` tells `gateslot.LoadConfig` over the hub's config dirs, and `standalonegeom.WebsterGeometry` the same loader over the worktree's own directory, so a standalone run reads the worktree's `gate.yaml` as the gate verb does outside every hub, and one with none is refused naming `lyx config gate` (card 2).
The verify gate's command seam, and `runVerify`, render `GateConfig().GateCall("final", "--all")`, `lyx gate test --wait --tier <final list> --all`, and run it through `verifytree.Verify` under the same site labels; the rendered command is the verified-tree record's key, so Merge-In skips on a tree this gate passed when `final` and `landing` hold the same list (card 14).
A config load error or a `TierList` refusal is the gate's returned error at that arrival, naming `lyx config gate`, never a pass; the plan-has-no-verify warning path goes, since the gate no longer reads the plan's section.
The gate keeps its outcome check, dirty check, pre-fix head, rerun and flake logic; its rerun re-runs the same call.
`runVerify`'s help text names the `final` gate in place of the plan's section, and its no-section `verified: false` answer goes.

`doc.go`'s verify-gate paragraphs and `contracts/specs/webster-spec.md`'s verify-gate paragraph say the gate runs the `final` gate's rendered call.

Tests: `TestVerifyGate_PassesWithoutVerifying` keeps its outcome rows and swaps the no-section row for a gate-config refusal returned as the gate's error.
`newVerifyGateRepo` tells a `GateConfig` reader over a seeded `gate.yaml` whose `final` tier runs a shell stand-in, and the integration rows put the `lyx` built from the tree first on `PATH` through `lyxbin.PutFirstOnPath`, so they run the rendered call through this tree's `lyx gate test`; those tests are not parallel, naming that process-global state.
`TestGateParity_VerifyGate` builds both gates over the same seeded config, with the same `PATH` step.
`TestWebsterGeometry` of both geometry packages asserts the constructor sets `GateConfig`, reading a seeded file and, for the standalone one, refusing a worktree with none naming `lyx config gate`.

**ImpactSummary:** the webster gate and its self-check verb read `gate.yaml` instead of the plan; both geometry constructors tell the new reader, so every webster geometry carries it, and no state or record format changes.
