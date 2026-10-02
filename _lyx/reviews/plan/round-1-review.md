---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: _lyx/plan/05-rework-archive-generation.md:225
    summary: a status commit after any non-Done PR-Rework exit sweeps the archived copies, the classless record and the review-run deletions into HEAD early, so one round splits across commits, against D14 and two acceptance criteria
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/07-rework-gate-whole-plan.md:279
    summary: card 7 checks first_card in internal/loomshed, against the decision record's Planparser Sole-Parser constraint, and names no home for the new rework-first-card check ID
  - id: F3
    severity: LOW
    location: _lyx/plan/00-overview.md:38
    summary: the live-generation round rule is stated three ways; cards 12 and 13 drop the class condition card 9 applies, no text says which is authoritative, and card 5 never names the record.json class field the rubrics would have to cite
  - id: F4
    severity: NIT
    location: _lyx/plan/10-rework-routes-through-plan-review.md:372
    summary: card 10's stated bound for the skip seam omits D7's agent-written inputs and the seam-error fallback to a real review
---

### [BLOCKING] A status commit splits one rework round across commits

**Location:** `_lyx/plan/05-rework-archive-generation.md:225` (and lines 217–218, 229–231); `_lyx/plan/00-overview.md:41–44`.

**Issue:**
`statusCommitPathspec` (`internal/loomcli/wiring.go:64`) adds `loomengine.LoomReviewsDirRel()` and `loomengine.LoomDurableDirRel()` whenever each holds a file, and `gitrepo.StageAndCommit` runs `git add -- <pathspec>`, which stages deletions as well as additions (`TestCommitStatusSeam_Real_ArchiveRenameCommitsAdditionAndDeletion` pins exactly that).
Shed calls the status seam on every `persist` (`internal/shedengine/run.go`): after a Stuck (PR-Rework has no `on_stuck`, so a gate that exhausts its three attempts persists `blocked`), a Failed (a returned error, including card 5's own collision error mid-archive), a Paused, and the `running` write a resume makes before re-calling the producer.
Within one uninterrupted `prRework.Call` no persist happens, so the happy path does make one commit.
On every other path, the status commit that lands between card 5's archive step and its round commit stages `_lyx/loom/rework/round-<N>/` whole (`findings.md`, the classless `record.json`, every `prior-generation/` copy) and the deletion of `_lyx/reviews/plan` and `_lyx/reviews/webster` (the discussion run directory still holds files, so the reviews root stays in the pathspec).
It does not stage the `_lyx/plan/` deletions or the webster durable directory's deletions, which wait for the round commit.
So D14's "the deletions and the archive land in one commit" and the acceptance criteria "one commit records both sides" and "a crash ... after the archive and before the session ... converge[s] to ... one commit" fail on exactly the interrupted paths those criteria exist for.
Card 5's crash-resume tests use fake seams and its integration test drives `Call` directly, so neither would see it.
The overview's `committed-round-carries-a-class` decision names the sweep but only guards `committedRejections` against it; nothing keeps the archive out of the status commit.

Verdict on card 7's companion claim: it survives.
`_lyx/plan/` is never in the status pathspec, so HEAD's plan stays the retired generation until the round commit, and `NextReworkCardNumber` over HEAD stays stable for the gate's window.

**Fix:**
Hold the in-flight archive out of the status commit.
Add a card that gives `loomshed` a working-tree predicate (the highest round directory has no `record.json` carrying a class) and narrows `statusCommitPathspec`: while a rejection is pending (`loomengine.LoomRejectionPath` exists) and that predicate holds, the status commit stages the status file and the drive reports only, leaving the reviews root and the loom durable directory to the round commit.
Once the round commit writes the class or the rejection is cleared, the next status commit sweeps both again, so nothing is lost, only deferred.
Pin it with a `wiring_commitstatus_test.go` table case and a real-git case in `wiring_commitstatus_integration_test.go` (archive moved, rejection pending, status commit stages nothing of the archive).
Reword the overview's `committed-round-carries-a-class` rationale and card 5's restatement of it: with the window held out, the classed-only count remains the rule for a classless record that reaches HEAD outside that window.

### [MEDIUM] The `first_card` comparison sits outside `internal/planparser`

**Location:** `_lyx/plan/07-rework-gate-whole-plan.md:279`.

**Issue:**
The decision record's Constraints state: "Planparser Sole-Parser Invariant: `first_card` is parsed and checked only in `internal/planparser`."
Card 7 adds the `rework-first-card` check in `internal/loomshed/gates.go`, comparing `plan.FirstCard` with `NextReworkCardNumber`.
Reading the model is allowed by `CONSTRAINTS.md`'s own wording, but the decision record narrows it for this key, and card 1 even restates "parsed and checked only here".
The card also adds a check ID without saying where it is documented.
`contracts/specs/loom-plan-spec.md`'s "Validation checks" section is the authoritative list for `planparser`'s IDs, and `internal/planglyph/doc.go` holds the canonical list of resolve-backed IDs; `rework-first-card` lands in neither, and card 1 deliberately keeps the twenty-eight-ID set unchanged.

**Fix:**
Put the comparison in `internal/planparser` as a function outside both format entry points (for example `CheckFirstCard(plan *Plan, told int) []ValidationError`, check `rework-first-card`), add a `planglyph` entry point that runs `ValidateFormat` plus that check converted blocking, and have `ValidateReworkPlan` compute the told number and call it.
Document the ID in `loom-plan-spec.md` beside the validation-checks list, as a rework-gate-only check outside both `ValidateFormat` and `Validate`, so the twenty-eight-row list and every count naming it stay unchanged.

### [LOW] The live-generation round rule has three wordings

**Location:** `_lyx/plan/00-overview.md:38–44`; `_lyx/plan/12-plan-review-rework-answer-key.md:422`; `_lyx/plan/13-webster-review-rework-range.md:448`; `_lyx/plan/05-rework-archive-generation.md:215`.

**Issue:**
`generation-round-rule` says Go and the agents apply "this one rule": the highest round whose `record.json` `first_card` matches.
`committed-round-carries-a-class` adds a condition (the record carries the class), which card 9 applies ("a classless record ... never skip[s]").
Cards 12 and 13 tell the agents the first form only, over working-tree directories.
In practice the reviews run after the round commit, when every round is classed, so the three pick the same round; but the plan never says so or names one statement as authoritative.
The rubrics also cannot name the condition, because card 5 never names the field that carries the class in `record.json`.

**Fix:**
State the rule once in the overview with the class condition, and say why the agents' working-tree reading agrees with card 9's HEAD reading (Plan-Review and Webster-Review run only after the round commit).
Name the field (`class`, values `exempt`/`required`) in card 5, and have cards 12 and 13 include "whose `record.json` carries a `class`" in the rule they write into the rubrics.

### [NIT] Card 10's bound statement is thinner than D7's

**Location:** `_lyx/plan/10-rework-routes-through-plan-review.md:372`.

**Issue:**
Card 10 introduces the routing edge and the `skip_seam` key, and its recipe comment states the bound as "only a generation whose round record and live cards both classify exempt skips, and generation 0 never does".
D7's bound also says the seam's inputs are agent-written (the overview's `first_card`, a swept `record.json`) and that re-classifying the live cards caps what a forged record can skip; card 8 adds that a failing seam falls back to a real review.
Card 9 states the first part and card 8 the second, so nothing is missing from the plan, but the recipe comment an operator reads carries neither.

**Fix:**
Have card 10's recipe comment say both: the inputs are agent-written, so the live cards are re-classified and a forged record can at most skip a generation whose live cards are themselves exempt; a seam error falls back to a real review.

## Focus verdicts

- Card 5 status-commit sweep: F1. Card 7's HEAD claim survives; card 5's commit description and its one-commit integration assertion hold only for an uninterrupted round.
- Card 7 placement and check-ID documentation: F2.
- Generation-round rule in three places: F3.
- (a) Recipe coverage guard: not a defect.
  `internal/loomrecipe/coverage_guard_test.go` maps rows to engines and never reads `on_done`; no row is added, renamed or re-engined, so it holds unchanged, and card 10 rewrites the header comment.
  (b) Plan-Review bounce budget on re-entry: not a defect.
  `episodeStuckCount` (`internal/shedengine/run.go:486`) ends a segment row's episode at any Done by a peer in its segment, pinned by `internal/shedengine/budgetexempt_test.go` since `b0359f2c8`; generation 0's approved Plan-Bouncer Done ends the episode before a rework generation re-enters, and card 10 says so.
- Card 5 granularity: not a defect.
  The archive step, the session's contract (stencil and spec factory) and the classed-only round count change together: the stencil's "write a whole plan into the empty `plan_dir`" is wrong without the archive, and the archive's early `record.json` is unsafe without the classed-only count.
  Deleting `CheckAppendOnly` removes the only callee of the code card 5 deletes; a separate card would be a one-file deletion with no test of its own.
  Card 4's split worked because it changed no behaviour; none of card 5's parts can be split that way.
  The status-commit change F1 needs is independently testable and goes in its own card, not into card 5.
- Card 8 and card 10 bounds against D7: F4 (card 10). Card 8's bound and fallback match D7.
  Card 10's `wire()` construction matches card 9's signature: `Env.Rework` is the whole `PRReworkDeps` value `wire()` builds, and the fields `PlanReviewSkippable` reads (`PlanDir`, `ReworkDir`, `ReworkDirRel`, `ReadCommitted`) survive card 5's reshaping.
