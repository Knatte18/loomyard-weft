# Card 3 — drop-parent-review-gate

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/specs/shed-recipe-spec.md`
- `internal/shedrecipe/doc.go`
- `internal/shedrecipe#resolveGateSpec`
- `internal/shedrecipe#resolveRowGateSpec`
- `internal/shedrecipe#resolveGateEntry`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#discussionWriteEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#shedrecipeAllowedImports`
- `internal/shedrecipe#TestResolveGateSpec_Rejections`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#discussionCommitPathspec`
- `internal/loomcli/doc.go`
- `internal/loomengine/discussion.go`
- `internal/loomengine#TestDiscussionTable`
- `internal/hubgeom/doc.go`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`
- `internal/loomrecipe#testEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/shuttleengine#GateEntry`

**Delete:**
- `internal/shedrecipe#parentReviewGateName`
- `internal/shedrecipe#resolveParentReviewClosure`
- `internal/shedrecipe#parentReviewEnv`
- `internal/shedrecipe#parentReviewCfg`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewResolvesMustPassMayHold`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewCapEqualsAttempts`
- `internal/shedrecipe#TestResolveGateSpec_ParentReviewMissingEnv`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsFiveValues`
- `internal/shedrecipe#TestDiscussionWriteEntry_ParentReviewBeginRound`
- `internal/shedrecipe#hasRound`
- `internal/loomcli#newParentReviewConfig`
- `internal/loomcli#keepAliveMargin`
- `internal/loomcli#keepAliveInterval`
- `internal/loomcli#parentWorktreeReviewerLive`
- `internal/loomcli#strandLive`
- `internal/loomcli/alignmentwiring.go`
- `internal/loomcli/alignmentwiring_test.go`
- `internal/loomcli/parentreview_wiring_test.go`
- `internal/loomcli/parentreview_integration_test.go`
- `internal/loomcli/smoke_parentreview_test.go`
- `internal/loomrecipe#testParentReviewConfig`

**Create:**
- `plan:internal/shedrecipe#TestResolveGateSpec_UnknownNameListsTheLegalValues` -> `func TestResolveGateSpec_UnknownNameListsTheLegalValues(t *testing.T)`

**Intent:** D1 and D2: Discussion-Write hands a passing discussion straight to Discussion-Bouncer, with no parent-review round, alignment spawn or parent message.

- `loom-recipe.yaml`: Discussion-Write's `gates` holds only `discussion` (attempts 3); drop the `parent-review` entry and its comment block.
  The row keeps its name and position, so an in-flight run waiting in a parent-review round under a driver on this binary passes on its next gate evaluation, and its round directory stays on disk unread; a driver on the old binary keeps the old gate until it restarts.
- `internal/shedrecipe`: the gate vocabulary is `discussion`, `plan`, `description` and `verify`.
  Delete `parentReviewGateName` and `resolveParentReviewClosure`, the last-element check in `resolveRowGateSpec`, the `parent-review` branch of `resolveGateEntry` (every element now reads `pass_on_cap` and resolves a plain closure), and the parent-review fresh-spawn preparation in `discussionWriteEntry`, which now passes a nil preparation.
  Remove the `ParentReview` field from `Env`, `internal/parentreview` from the production imports and from `shedrecipeAllowedImports`, and every parent-review sentence from `resolveGateSpec`'s doc, `discussionWriteEntry`'s doc and `doc.go`.
  The unknown-name refusal in `resolvePlainGateClosure` names the four legal values.
- Tests: delete the parent-review tests and helpers listed, drop the `parent-review` rows of `TestResolveGateSpec_Rejections`, and replace `TestResolveGateSpec_UnknownNameListsFiveValues` with `TestResolveGateSpec_UnknownNameListsTheLegalValues`, which asserts an unknown gate name is refused naming `discussion`, `plan`, `description` and `verify`, and that `parent-review` is now such an unknown name.
- `internal/loomcli`: `wire` builds no parent-review config or aligner and sets no `Env.ParentReview`; delete the helpers listed and `alignmentwiring.go`, whose `alignment` use is the last outside the package bar `orchengine`'s, moved by card 1.
  `discussionCommitPathspec` stages the discussion directory and the amendment record only, since nothing writes a parent-review round any more; an existing round directory stays committed as it is.
  The package doc's `# Parent-review alignment` section goes.
  The criterion "Discussion-Write's only gate is `discussion`" is pinned by `TestSequence_FullRunBlocksAtMergeIn`'s existing table-gate assertion, which now wants `[]string{"discussion"}`; the deleted parentreview wiring, integration and smoke tests pinned the removed gate.
- `internal/loomengine/discussion.go`: delete `DiscussionWriterRole`, whose one caller was the keep-alive lifetime, and its header sentence; `TestDiscussionTable` checks the chair's role against `"discussion"` alone.
- Test fixtures: `buildSequenceFixture`, `testEnv` and `envkit`'s `nilLegal` table drop their `ParentReview` entries.
- In-flight criterion: `buildSequenceFixture` also lays out, under the anchor, an open parent-review round as an older binary left it (`_lyx/reviews/parent-review/round-1/request.json` with `state: open` and no `verdict.json`), and `TestSequence_FullRunBlocksAtMergeIn` asserts Discussion-Write's history entry is Done to Discussion-Bouncer and the round directory is still on disk unchanged, so a run waiting in a parent-review round passes Discussion-Write on its next gate evaluation.
- `internal/shuttleengine/gate.go`: `GateEntry.MayHold`'s doc sentence "Only the parent-review wiring sets it" says no shipped gate sets it, since the parent-review entry was its one setter.
- Docs: `shed-recipe-spec.md` drops the `parent-review` vocabulary entry, its last-element rule and its paragraph, and the Discussion-Write preparation sentence; `hubgeom/doc.go` drops the Discussion-Write parent review from what the parent names.

What it lets through: a discussion that passes its `discussion` gate reaches Discussion-Bouncer with no parent check; the bound is the Discussion-Review perch, whose `review-discussion` fan carries the scope lenses after card 7 and whose Bouncer routes `decision` findings to the parent after card 12.

**ImpactSummary:** Removes one gate kind, one Env field and the loom wiring behind them; every caller is in shedrecipe, loomcli, loomrecipe and the envkit fixture, plus one shuttleengine doc sentence.
