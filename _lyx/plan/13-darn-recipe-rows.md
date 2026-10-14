# Card 13 — darn-recipe-rows

**Create:**
- `plan:internal/loomrecipe#TestDarnPerch_ConvergedOverAnOpenDecisionFindingAcceptsAndCarriesItOver` -> `func TestDarnPerch_ConvergedOverAnOpenDecisionFindingAcceptsAndCarriesItOver(t *testing.T)`
- `plan:internal/loomrecipe#decisionOpenConvergedBouncerJudge` -> `func decisionOpenConvergedBouncerJudge(env shedrecipe.Env)`

**Edit:**
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe#TestBouncerRowsCarryOverTheirOwnSegment`
- `internal/loomrecipe#TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe`
- `internal/loomrecipe/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/loomrecipe#writeBouncerJudge`
- `internal/loomrecipe#designWrongBouncerJudge`
- `internal/loomrecipe#haltingProducer`

**Intent:** Change the darn recipe's rows: a `discussion` gate on Darn-Write, a carry-over on Darn-Bouncer, and a PR-Align row between Publish and PR-Gate (D4, D8, D9).
No row is renamed, and an in-flight darn run is restarted, not migrated (A4).

In `darn-recipe.yaml`:
the Darn-Write row's `gates` list gains `{name: discussion, attempts: 3}` first, ahead of `verify` and `description`, Discussion-Write's attempt count (A5), with a comment that on a darn run it is the darn gate, which also refuses a record missing an entry its history held and commits the two files when it passes;
the Darn-Bouncer row gains `carry_over: Darn-Review`, its own segment, and the comment saying a darn run has no record to file findings in goes;
Publish's `on_done` becomes `PR-Align`, and a new `PR-Align` row (engine `PRAlign`, `on_done: PR-Gate`, no segment, no `on_stuck`) sits between Publish and PR-Gate, with loom's comment that it always ends Done and sits outside the PR-Review segment because a Done there would end the gate's bounce episode;
the header comment says the writer writes and validates the decision record before any code, the perch's open findings land in the record's `## Open risks`, PR-Align spawns the alignment agent with darn's geometry before PR-Gate and reads loom.yaml's alignment keys, an in-flight run on the changed edges is restarted, and PR-Align, like Friction-Reflect, carries no `on_stuck` because it never returns Stuck.
With `review_max_bounces: 0`, `applyDarnReviewBudget` still routes Darn-Write's Done to Merge-In.
PR-Gate keeps `on_stuck: Darn-Write`: an agent reject writes the rejection record, PR-Gate bounces to Darn-Write, and `loomshed`'s Darn-Write renders it as rejection findings, the path `TestDarnWrite_Call`'s rejection row and `TestPRAlign`'s first-reject and second-reject rows already pin recipe-neutrally.

`wantDarnProducerTable` gains `{loomshed.NamePRAlign, "PRAlign", reflect.TypeOf(&landingshed.PRAlign{})}` between Publish and PR-Gate, which `TestNewDarn_RowsMatchTheDarnTable` and `TestDarnRouting` then check.
`TestNewDarn_Routing` asserts Publish's `OnDone` is PR-Align and PR-Align's is PR-Gate with an empty `OnStuck` and no segment, keeps PR-Gate's bounce to Darn-Write and the three reroute edges, and its budget-0 subtest still wants Darn-Write's `OnDone` to be Merge-In.
`TestDarnVerifyBudget`'s want map gives Darn-Write `{"discussion": 3, "verify": 7, "description": 3}`.
`TestBouncerRowsCarryOverTheirOwnSegment` runs over both embedded recipes, so it pins `carry_over` equal to the segment on Darn-Bouncer too.
`TestEscalation_DarnPerchRuleContinuesThenHardCapAcceptReachesDescribe` swaps `env.CarryOver` for a capturing closure and, after the accept, asserts one carry-over entry whose segment is `Darn-Review` and whose findings hold the open `design-gap` entry, in place of its two no-decision-record assertions; its doc drops "with no decision record on disk".
`decisionOpenConvergedBouncerJudge`, in `fixture_test.go` beside `designWrongBouncerJudge`, makes every bouncer-judge round write a CONVERGED verdict over a ledger holding one open `decision`-class MEDIUM entry with no fix tag.
`TestDarnPerch_ConvergedOverAnOpenDecisionFindingAcceptsAndCarriesItOver`, in `escalation_test.go`, builds `NewDarn` with that judge and a capturing `CarryOver`, fakes Preflight and Darn-Write as Done and Describe with a `haltingProducer`, runs the shed once and asserts it reaches Describe with no circling escalation file in the run directory, and that the carry-over entry has segment `Darn-Review` and holds the `decision` MEDIUM finding.
The new top-level test raises `internal/loomrecipe`'s row in `cmd/lyx/testdata/test-budget.yaml` from 54 to 55.

`internal/loomrecipe/doc.go` drops "the darn recipe has no PR-Align row" and names darn's tail as `Describe → Merge-In → Publish → PR-Align → PR-Gate → Finalize`.
`contracts/specs/refusal-spec.md`'s `Darn-Write verify gate exhausted` row covers the discussion gate too: its condition names a discussion gate still failing after its recipe attempts.
`docs/overview.md`'s darn paragraph says Darn-Write writes and validates the decision record first and its discussion gate commits it, the perch measures against the board entry as amended by the record's added entries and carries its open findings into the record, and the darn tail reads `Describe → Merge-In → Publish → PR-Align → PR-Gate → Finalize`.

**ImpactSummary:** the darn recipe gains a gate entry, a carry-over key and a row; the loomrecipe darn tests and the docs that describe darn's rows follow, and a loom run is untouched.

**Commit:** `13: the darn recipe gates the record, carries over the perch's findings and aligns the PR`
