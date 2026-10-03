# Card 20 — shuttlefake-webster

**Edit:**
- `websterengine/strand_test.go`
- `websterengine/runlevel_test.go`
- `websterengine/recordbatch_test.go`
- `websterengine/recoverbatch_test.go`
- `websterengine/beginbatch_test.go`
- `webstercli/verbs_test.go`

**Uses:**
- `testkit/shuttlefake/shuttlefake.go`

**Intent:**
Move the webster pair's `Engine` and `ReedOps` fakes onto `shuttlefake` (#324 §1).

Delete `fakeReed`/`fakeEngine` (`strand_test.go`), `runFakeReed`/`runFakeEngine`, `recordFakeEngine`, `recoverFakeReed`/`recoverFakeEngine`, `beginFakeReed`/`beginFakeEngine` and `verbsFakeReed`/`verbsFakeEngine`; each test builds a `shuttlefake.Engine`/`shuttlefake.Reed` and sets the func fields or scripted values its old fake hard-coded.
Scripted answers keep their exact values and every assertion stays; a test that read a counter on the old fake reads the kit's recorded calls.
A fake carrying behaviour beyond the seam (`beginFakeInjector`, `verbsFakeMasterStarter`, the starters and clocks) is not an `Engine`/`ReedOps` fake and stays.

`strand_test.go` is untagged and `shuttlefake` spawns nothing, so it stays untagged.
Apply D10 to every touched test.

**ImpactSummary:** Webster test files only (one untagged, the rest integration-tagged); local Engine/ReedOps fakes deleted in favour of the kit, assertions unchanged.
