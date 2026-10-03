# Card 33 — loomrecipe-envkit

**Edit:**
- `loomrecipe/fixture_test.go`
- `loomrecipe/shape_test.go`
- `loomrecipe/sequence_test.go`
- `loomrecipe/resume_test.go`

**Uses:**
- `testkit/envkit/envkit.go`
- `testkit/shedfake/shedfake.go`

**Intent:**
Replace loomrecipe's copied builders and fakes with the kits (#324 §2, §3).

`fixture_test.go`: `fakeMergeShuttle`, `nilFabricOpener`, `testLandingDeps`, `testReworkDeps` and the four placeholder seams go, in favour of `envkit.LandingDeps`, `envkit.ReworkDeps` and `shedfake.WebsterSeams()`.
`fakeLoomShuttle`'s gate methods and `evalGateList` go: it becomes a `shedfake.Shuttle` whose `RunFn` keeps the per-producer dispatch (and the bouncer-judge branch) the sequence tests script, so every scripted answer is unchanged.
`fakeLoomBurler` becomes `shedfake.BurlerRunner` if its behaviour is the kit's scripting; a per-profile dispatch it needs stays as a `DuringRun` hook, not a type.
`buildSequenceFixture` starts from `envkit.FullEnv(t)` and overrides what the sequence tests need (anchor, paths, the loom shuttle, the webster runner).
Helpers that serve only this package (`writeDiscussionFixture`, `seedPlanFixture`, `writeBatcherConfig`, `seedBouncerStencils`, `fakeWebsterRun`, `fakeAlwaysDoneProducer`) stay.

`shape_test.go`'s `testEnv` and its inline rework-deps literal start from `envkit.FullEnv`.

The header comment of `fixture_test.go` that cites the "duplicate-test-helpers-rather-than-share-them Shared Decision" is rewritten to describe the file's remaining helpers, without the policy reference (D1).
Apply D10 to every touched test.

**ImpactSummary:** Untagged loomrecipe tests only: two fixture files drop their copied builders and fakes for the kits, and the sequence and resume tests that construct the loom shuttle and burler fakes switch to the kit types; sequence-test scripting is preserved.
