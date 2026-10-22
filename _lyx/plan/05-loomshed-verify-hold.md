# Card 5 — the loomshed verify gate tells its scope's pause hold

**Edit:**
- `internal/loomshed#NewVerifyGate` -> `func NewVerifyGate(worktreeRoot, verifyDir, siteLabel string, command, mergeBase func() (string, error), treeScope func() (string, bool, error), round func() (RoundGate, error), hold *verifytree.PauseHold) shuttleengine.Gate`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/loomcli#loomCLI.wire`
- `internal/loomshed#TestVerifyGate_Scenario`
- `internal/loomshed#TestVerifyGate_RoundScenario`
- `internal/loomshed#roundGateFixture.runGate`
- `internal/loomshed#roundGateFixture.diffGate`
- `internal/webstercli#TestGateParity_VerifyGate`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#FullEnv`

**Uses:**
- `internal/verifytree#NewPauseHold`
- `internal/verifytree#PauseHold`

**Intent:**
D5 for the loomshed verify gate, the gate of the Darn-Write, Darn-Burler-Fix and Webster-Burler-Fix rows.
`NewVerifyGate` takes a `hold` and passes it as `Site.Hold` on its one `verifytree.Verify` call; a nil hold runs as today.
Its doc comment, which names each parameter, says a non-nil hold holds the verify under a pause covering the worktree.
`shedrecipe.Env` gains the field `VerifyHold *verifytree.PauseHold`, documented as the told pause hold of the worktree's verify sites, read by the gate resolver's "verify" gate, nil holding nothing; `resolvePlainGateClosure`'s "verify" case passes `env.VerifyHold`, and its doc names it.
`loomCLI.wire` sets `VerifyHold: verifytree.NewPauseHold(pauseGeom, nil)` in the `shedrecipe.Env` it builds, from the pause geometry it already resolves for the runner, so loom's Webster-Burler-Fix row and, through `wireDarn`, which only adds fields to that env, the darn rows tell the hold of the worktree's slug.
None of these rows holds a parent lock, so the hold exempts no slug entry.
The tests that call `NewVerifyGate` pass a nil hold.
`TestWire_EverySeamFilled` already fails on a nil `Env` pointer through `envkit.NilSeams`, so it pins that `wire` fills `VerifyHold`; its doc comment names the field.
Every other `NilSeams` check over an `Env` sees the new pointer too: battencli's env leaves it nil, so `VerifyHold` joins battencli's `intentionallyNil` with the reason `loom-only seam, batten runs no verify gate`, and `envkit.FullEnv` fills it with `verifytree.NewPauseHold(pauserecord.Geometry{}, nil)`, which never returns nil and holds nothing, so `TestFullEnv_FillsEverySeam` stays green.

**ImpactSummary:** one exported constructor re-signed with its one production caller (`shedrecipe`) and four test callers in loomshed and webstercli; one new `Env` field filled by loom's wiring, reaching darn through the same env.
