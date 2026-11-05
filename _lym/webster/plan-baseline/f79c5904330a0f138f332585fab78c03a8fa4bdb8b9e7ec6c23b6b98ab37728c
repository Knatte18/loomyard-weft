# Card 6 — remove-reopen

**Delete:**
- `internal/beamcli#reopenCmd`
- `internal/beamcli#reopen`
- `internal/beamcli#reopenRows`
- `internal/beamcli#TestReopen`
- `internal/beamcli#TestHubLandDeps_UnreadyCommitsTheRemovalOfTheReadyRecordAndSucceedsAgain`
- `internal/pawlengine#ReopenRequest`
- `internal/pawlengine#WriteReopenRequest`
- `internal/pawlengine#ReadReopenRequest`
- `internal/pawlengine#RemoveReopenRequest`
- `internal/pawlengine#awaitRun.answerReopen`
- `internal/pawlengine_test#world.reopen`
- `internal/landingshed#ReopenRequest`
- `internal/landingshed#PRReview.answerReopen`
- `internal/landingshed#reopenDraftComment`
- `internal/landingshed#reopenCloseComment`
- `internal/landingshed#TestPRReview_Reopen`
- `internal/loomcli#readReopenRequest`
- `internal/loomcli#TestReadReopenRequest`
- `internal/loomshed#NameBeamDescribe`

**Edit:**
- `internal/beamcli#Command`
- `internal/beamcli#landDeps`
- `internal/beamcli#hubLandDeps`
- `internal/beamcli#onHubLandDeps`
- `internal/beamcli#landWorld`
- `internal/beamcli#landWorld.deps`
- `internal/beamcli#newLandWorld`
- `internal/beamcli/doc.go`
- `internal/pawlengine#Paths`
- `internal/pawlengine#Deps`
- `internal/pawlengine#awaitRun`
- `internal/pawlengine#awaitRun.enter`
- `internal/pawlengine/doc.go`
- `internal/pawlengine_test#TestBeamAwait`
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.decide`
- `internal/landingshed#PRReview.wake`
- `internal/landingshed/doc.go`
- `internal/loomcli#pullRequestDepsGuardedBy`
- `internal/loomcli#intentionallyNil`
- `internal/loomcli/doc.go`
- `internal/battencli#intentionallyNil`
- `internal/testkit/envkit#LandingDeps`
- `internal/hubgeom#PawlPaths`
- `internal/loomshed/doc.go`
- `docs/overview.md`

**Uses:**
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** No way back from ready (D10): `lym beam reopen`, the reopen request, Beam-Await's handling of it and PR-Review's reroute to Beam-Await go.

- `beamcli`: the `reopen` verb, its function, `reopenRows`, `TestReopen` and the `WriteReopen` and `Unready` seams of `landDeps` with their `hubLandDeps` closures go; `Command` no longer adds it and its Short and Long no longer name reopen; the `onHubLandDeps` doc names the verbs left. `newLandWorld` drops the two fakes. The integration test of `Unready` goes with the seam. The file header comments of `internal/beamcli/land.go` and `internal/beamcli/deps.go`, which name `lym beam reopen` and the reopen request, are reworded to the verbs and seams left.
- `pawlengine`: the `ReopenRequest` type and its write, read and remove functions go, and Beam-Await's entry no longer reads one (`answerReopen` and `reopenStuckNoticed` go from `awaitRun`; `enter` no longer calls it). `ReopenRequestFileName` and `Paths.ReopenRequest` stay, documented as the reopen request an older binary wrote, which card 8's retired-row move removes; `pathsSeams` keeps requiring it. `TestBeamAwait` drops its reopen rows and the world's `reopen` helper goes.
- `landingshed`: `PullRequestDeps.Reopen`, the `ReopenRequest` type, `PRReview.answerReopen`, its two comment constants and its calls in `decide` and `wake` go, so PR-Review never reroutes on a reopen; the PR-Review doc paragraphs on the reopen go, while the paragraph that marks a found draft ready for review stays, since tapestry still meets drafts. `TestPRReview_Reopen` goes.
- `loomcli`: `readReopenRequest`, its test and the `Reopen` wiring go; the doc's pawl row list names the new rows. `intentionallyNil` in loomcli and battencli drop `Landing.PullRequest.Reopen`; envkit's `LandingDeps` drops its fake.
- `loomshed.NameBeamDescribe` goes: no row and no code names it after card 5 and this card; the retired names live in the pawl recipe's `retired` map.
- `docs/overview.md`'s beam bullet drops `lym beam reopen` and states that `lym beam ready` is final.
- Bound: `lym loom goto` stays the operator's hand escape; card 10 gives a goto back to Beam-Await under a standing ready record its entry rule, and card 7's guards start no member while the ready record stands, so the goto adds no way to take new work.

Tests (tier 0): the removed tests go with their subjects; `TestBeamAwait` keeps every other row passing; the seam tests listed under Uses keep passing with the field gone, since each reflects over the struct as it stands.

**ImpactSummary:** removes one verb, one request file format and one PR-Review branch used only by pawl; every caller is in beamcli, pawlengine, landingshed, loomcli and their test fixtures, all listed.

**Commit:** 6: Remove lym beam reopen and the reopen request
