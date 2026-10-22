# Card 6 — the webster verify gate and lyx webster verify tell their pause hold

**Edit:**
- `internal/websterengine#Geometry`
- `internal/websterengine#NewVerifyGate`
- `internal/websterengine/doc.go`
- `internal/webstercli#websterCLI`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#websterCLI.runVerify`
- `internal/webstercli#TestWire_ModeHubSelectsHubMode`
- `internal/webstercli#TestWire_StandaloneMode`

**Uses:**
- `plan:internal/verifytree#NewPauseHold`
- `plan:internal/verifytree#PauseHold`

**Intent:**
D5 for the webster verify gate and the `lyx webster verify` verb.
`websterengine.Geometry` gains the field `VerifyHold *verifytree.PauseHold`, documented as the told pause hold the verify gate passes to every Verify, nil holding nothing; `hubgeom` and `standalonegeom` leave it nil, and the CLI layer sets it, as it sets `Index`.
`NewVerifyGate`'s `verify` seam sets `site.Hold = geom.VerifyHold` before its `verifytree.Verify` call, so the gate's first run and its flaky rerun both hold; its doc comment and the verify-gate paragraph of `doc.go` say so.
`websterCLI` gains the field `pause pauserecord.Geometry`, the pause geometry its wiring resolved, read by `runVerify` here and by card 16's batch verbs.
`wireHub` stores the `pauseGeom` it already resolves for the runner in `c.pause` and sets `geom.VerifyHold = verifytree.NewPauseHold(pauseGeom, nil)` before `c.geom = geom`; `wireStandalone` does the same with `standalonegeom.PauseGeometry(res.StateDir)`.
`runVerify` passes `Site{Label: verifyVerbSite, Hold: c.geom.VerifyHold}`, so the verb, which runs under the operator's or Master's own context, holds like the gate; its `Long` text says a pause holds the run until the resume.
`TestWire_ModeHubSelectsHubMode` and `TestWire_StandaloneMode` each assert the wired `c.geom.VerifyHold` is non-nil and `c.pause` names the record path the runner was told.

**ImpactSummary:** one new told field on webster's geometry read by its verify gate, two wiring paths that fill it, and the verify verb; no other reader of `Geometry` changes.
