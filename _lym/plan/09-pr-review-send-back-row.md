# Card 9 — pr-review-send-back-row

**Create:**
- `plan:internal/landingshed#PRReview.settleSendBack` -> `func (p *PRReview) settleSendBack(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, bool, error)`
- `plan:internal/landingshed#PRReview.sendBack` -> `func (p *PRReview) sendBack(ctx context.Context, rec pullrequest.Record) (shedengine.Outcome, shedengine.OutputPointer, bool, error)`

**Edit:**
- `internal/landingshed#PullRequestDeps`
- `internal/landingshed#PRReview.Call`
- `internal/landingshed#PRReview.checkWiring`
- `internal/landingshed#PRReview.wake`
- `internal/landingshed#TestPRReview_Call`
- `internal/landingshed#TestPRReview_Wait`
- `internal/landingshed#TestPRReview_Session`
- `internal/landingshed#newPRReviewFixture`
- `internal/landingshed#newWaitWorld`
- `internal/landingshed#prReviewTestDeps`
- `//internal/landingshed/doc.go`
- `internal/loomcli#pullRequestDeps`
- `internal/testkit/envkit#LandingDeps`

**Uses:**
- `internal/landingshed#stuckVerdict`
- `internal/landingshed#PRReview.stopLiveSession`
- `internal/shedengine#HistoryEntry`
- `internal/shedrun#StatusFile`
- `internal/landingshed#fakePRSessions`
- `internal/shedengine#TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore`
- `//internal/loomcli/doc.go`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Intent:** D4's PR-Review side: the row reroutes a marked send-back to Code-Bouncer, and clears the mark only once shed has persisted that reroute.

- `PullRequestDeps` gains `ShedStatus func() (shedengine.Status, error)`, the run's shed status, since a producer's `Call` receives no history; `checkWiring` requires it.
  `loomcli`'s `pullRequestDeps` reads `shedrun.StatusFile` under its lock, as `samplerDoneCmd` does; `envkit`'s `LandingDeps` fills it.
- `Call` runs `settleSendBack` after `checkWiring` and before `decide`, so before any session starts or resumes.
  With no mark it does nothing.
  With a mark, it reads the row's newest own history entry.
  When that is an `OutcomeReroute` entry whose `Output` is the mark's file, the reroute persisted:
  it clears the mark, `FixHereCount` with its `CountedSets`, `ParentReviewAsked` and the session fields `StrandGUID`, `StrandName`, `SessionID` and `Engine` in one record write, commits the record, and the row runs as on any entry, starting a new session and asking the parent again with `pr_review_ask_parent` on.
  With any other newest entry it keeps all of them and runs `sendBack`: a re-step, a crash before the persist, the resume after the dirty-tree Stuck, the resume after a reroute-cap halt and a `goto` back to the row.
  Matching the file, not only the outcome, keeps a crash after a later send-back from taking an earlier send-back's reroute for its own.
- `wake` runs `sendBack` when the record it reads holds a mark, before any gate.
- `sendBack` stops the live session through `stopLiveSession`, then reads the tree again, since the agent may have changed it after the verb answered.
  On a clean tree it returns `Stuck` with `Reroute` set and `Path` the mark's file, no `Halt`, and clears nothing; shed persists the reroute entry with that file as its output.
  On a dirty tree it halts Stuck for the parent with `pull request #<n> has findings sent back to Code-Review, but the task worktree has uncommitted changes: <paths>; way forward: commit or discard them in the task worktree, then run lym loom resume`; the resumed row meets the mark on a clean tree and reroutes.

The reroute cap bounds the edge as for any reroute: the cap's halt keeps the mark and the count, and its resume reroutes that send-back within the reroutes it grants.
Shed's generic cap, which holds for this edge with no PR-Review code, is pinned by `TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore`.

Tests, tier0, rows of the existing tests over their fixtures.
`prReviewTestDeps`, which builds the deps of every PR-Review test, fills `ShedStatus` from a scripted status, empty history by default, so `checkWiring` passes; `newPRReviewFixture` and `newWaitWorld` set the status per row, and `newWaitWorld` takes a `fakePRSessions`, whose stops a row can count.

- `TestPRReview_Call`: newest entry the marked file's reroute starts with no mark, count 0 and no session id (criterion D4);
  a re-step, newest entry the row's own Stuck, keeps the count and, on a clean tree, reroutes with the file as output;
  a re-step with no mark and a non-zero `FixHereCount` keeps the count;
  a mark with no history entry of the row's own (a crash on the first send-back before the persist) keeps the mark and the count and reroutes;
  newest entry a reroute of another file (a crash between a later send-back's verdict and its persist) keeps the mark and the count and reroutes;
  newest entry a `RerouteCapped` Stuck keeps them and reroutes;
  a dirty tree halts with the message above.
- `TestPRReview_Wait`: a wake whose record carries the mark stops the session, counted on the `fakePRSessions`, and returns the reroute;
  a wake whose tree is dirty after the stop halts with the dirty-tree message, and the next Call on a clean tree reroutes.
- `TestPRReview_Session`: after the clear, the row starts a new session and posts a new parent review request with the setting on.

**ImpactSummary:** PR-Review's Call and wake gain the send-back mark's handling over one new required seam, wired in loomcli and envkit; a run with no mark behaves as before.

**Commit:** `9: pr-review-send-back-row`
