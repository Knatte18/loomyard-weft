```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked three things: test seams and tier tags, doc text the card makes untrue, and whether the card can pass its gate in plan order.

**Per-card list**

- **Card 1**
  - Seams: none; held.
  - Tier: no new test file; held.
  - Docs: `doc.go`, the `tasksetting.go` file comment and `pattern/PATTERN-config-layers.md` are all on the card; held.
  - Order: `LookupAutonomy` has no production caller until card 3; at risk under the deadcode guard (finding 3).
- **Card 2**
  - Seams and tier: held.
  - Docs: the `configChangeNotice` comment is on the card; held.
  - Order: uses card 1's key; held.
- **Card 3**
  - Tier: the new tier1 file is named; held.
  - Docs: `hubgeom/doc.go` is extended; held.
  - Order: `Autonomy` has no production caller until card 4; at risk (finding 3).
- **Card 4**
  - Tier: tier1 is named; held.
  - Docs: `orchcli/doc.go` is in `Uses:` only. Nothing I read there is made untrue; held, not verified.
- **Card 5**
  - Seams: held.
  - Docs: broken in a minor way. `burlerengine/doc.go:52` says an ask moves only to an end state. The card adds a `Cleared` mark written on an ended ask, but lists `doc.go` under `Uses:` only (finding 5).
  - Order: `ClearParentAsk` has no production caller until card 6; at risk (finding 3).
- **Card 6**
  - Seams: `driftengine.Config.Autonomous` is no seam; held.
  - Docs: broken (finding 1).
  - Order: uses cards 3 and 5; held.
- **Card 7**
  - Seams and tier: held. `TestRunAutonomy_ForcesTheRunsKeysOverARealHub` is named tier1.
  - Order: uses card 6's `runAutonomy`; held.
- **Card 8**
  - Seams: held.
  - Order: `ClearReviewSession` has no production caller until card 14; at risk (finding 3).
- **Card 9**
  - Seams and tier: held.
  - Order: `PRReviewAsker.Ask`, `PRReviewAskLine` and `ReadPRReviewStanding` have no production caller until card 10; at risk (finding 3).
- **Card 10**
  - Seams: held. The three `PullRequestDeps` fields reach `optionalPullRequestSeams`, `envkit.LandingDeps`, both `intentionallyNil` lists and `TestLandingDeps_EveryFieldPopulated`.
  - Marker test: held. `TestStencils_TopLevelMarkers` does not pin the PR-Review stencil, so the new `review_mode` marker needs no row there.
  - Docs: `landingshed/doc.go` is edited; held.
- **Card 11**
  - Docs: `docs/overview.md` is edited, and `landingshed/doc.go` is already edited by card 10; held.
  - Order: the `PRReviewStanding` seam comes from card 10; held.
- **Card 12**
  - Seams: `parentReviewDeps` is not walked by reflection; held.
  - Docs: `loomcli/doc.go` and `docs/overview.md` are listed; held.
  - Order: `prReviewAsker` comes from card 10; held.
- **Card 13**: seams and docs held. Order: uses card 9; held.
- **Card 14**
  - Docs: broken. `shedengine/goto.go:70` says `Goto` "commits nothing", and the card does not list that comment (finding 2).
  - Order: uses card 8; held.
- **Card 15**
  - Seams: held. The `pawlengine.Deps` fields reach `envkit.FullEnv`, battencli's `intentionallyNil` and the `beamAwaitEntry` and `requirePawl` maps.
  - Docs: `pawlengine/doc.go` is edited; held.
- **Card 16**
  - Seams: no reflective walk of `slackCLI` was found; held.
  - Docs: `slackcli/doc.go` is edited; held.
- **Card 17**: no new marker in `role-autonomy.md`; held. The `orch-role` marker row is unchanged.
- **Card 18**
  - Seams: the new `orchengine.Paths.Autonomy` field has no reflective walk; held.
  - Docs: `orchengine/doc.go` is in `Uses:` only; held, not verified.

**Findings**

1. MEDIUM, consistency.
   - Location: `_lym/plan/06-drift-autonomy.md`, which lists `internal/driftengine/doc.go` under `Uses:` only.
   - Summary: card 6 makes the package doc untrue and does not edit it.
     - Under autonomy, an ask that times out or never reached the parent becomes a Drift-Verify Stuck naming `lym loom resume`.
     - The resume clears the ended ask and asks again over the same rulings.
     - `judge_ask_parent` and `judge_interactive` are forced.
     - The doc still says the parent-agreement entry "passes only on an answer to those rulings, a timeout, or an ask that never reached the parent". It describes the two keys only by their layered meaning, and the Drift-Verify Stuck list lacks the new halt.
   - Evidence: `internal/driftengine/doc.go:16` and `:90-93`.
   - Fix: move `internal/driftengine/doc.go` under `**Edit:**` on card 6 and state the halt, the resume re-ask and the forcing.
2. LOW, consistency.
   - Location: `_lym/plan/14-goto-clears-pr-review.md`, `**Edit:**`, which lists `internal/shedengine#GotoGuard` but not `internal/shedengine#Goto`.
   - Summary: card 14's reset clears and commits the pull request record from inside the goto guard. `Goto`'s own comment, "It sets no transient mark and commits nothing", is then untrue, and the card does not list it.
   - Evidence: `internal/shedengine/goto.go:70`.
   - Fix: add `internal/shedengine#Goto` and state the one exception there, or say in the `GotoGuard` sentence that a commit happens.
3. MEDIUM, design.
   - Location: `_lym/plan/08-pr-record-autonomy.md`; the same pattern holds for cards 1, 3, 5 and 9.
   - Summary: these cards create exported production functions whose first production caller is a later card:
     - `pullrequest.ClearReviewSession` is first called by card 14;
     - `burlerengine.ClearParentAsk` by card 6;
     - `dobbyengine.LookupAutonomy` by card 3;
     - `hubgeom.Autonomy` by card 4;
     - `landingshed.PRReviewAsker.Ask`, `PRReviewAskLine` and `ReadPRReviewStanding` by card 10.

     The guard `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` is tier1. It fails on any production function that no `main` reaches. Any batch boundary that splits a creator from its caller leaves a state that fails tier1. Card 8 to card 14 is the widest gap.
   - Evidence: `cmd/lym/testonlycode_linux_test.go:79-80`; `pattern/PATTERN-test-only-code.md`, Enforcement section.
   - Fix: give each new function its production caller on the same card, or move card 8 next to card 14. If the split stays, the plan must keep each creator and its caller in one batch.
4. NIT, consistency.
   - Location: `_lym/plan/05-parent-ask-clear.md`, `**Uses:**` `internal/burlerengine/doc.go`.
   - Summary: the doc says an ask moves only to an end state and never moves after. The card's new `Cleared` mark on an ended ask goes undocumented, and the card lists `doc.go` under `Uses:` only.
   - Evidence: `internal/burlerengine/doc.go:52`.

**Tool calls**

1. `pattern/PATTERN-test-only-code.md` and a search for its enforcing test. The manifest lacked the reachability invariant's enforcement.
2. The tier of `cmd/lym/testonlycode_linux_test.go` and a search for stencil verb scans. Needed to judge whether card order breaks a gated test.
3. The relevant sections of `driftengine/doc.go`, `shedengine/doc.go` and `burlerengine/doc.go`. These package docs were not in the manifest.
4. `driftengine/doc.go` judge and Stuck sections, `burlerengine/doc.go` parent-ask section, and greps of the landingshed and pawlengine docs. Needed to confirm which doc text the cards make untrue.
5. Cases of `contracts/stencils/markers_test.go`, reflective tests in slackcli and orchengine, and the "commits nothing" comment in `goto.go`. Needed for card 10's marker and the card 16, 18 and 14 seam and doc checks.
6. Whether `TestStencils_TopLevelMarkers` pins the PR-Review stencil. Needed for the card 10 marker invariant.
