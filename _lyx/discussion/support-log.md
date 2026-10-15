# Support log — twb-d0-sc

## Interview

Autonomous run: no operator answered, and the seat has no advisors; every question below was settled by the writer and is an auto-pick in the ledger.

### Turn 1 — Where does the Darn review read its scope today?

The board entry asks this first.
Darn-Bouncer and Darn-Burler-Review take the rubric `loom-rubric-darn-review.md`, which names the board entry, read live with `lyx board get`, as the measuring stick; Darn-Burler-Review's `fasit` in `darn-recipe.yaml` says the same.
No copy of the body exists, so the change touches the rubric and the fasit text only.

### Turn 2 — Why did darn leave PR-Align out?

`a40c7baef`'s message says only "darn's recipe is unchanged".
The alignment agent at `pr-gate` reads the decision record, the plan and the open risks Merge-Review carries over; darn had none of them.
`landingAlignmentDeps` already builds PR-Align's seams for every recipe, so the row is a recipe change plus a darn prompt input.

### Turn 3 — Batch: record shape, gate, decision verb

- Same path and format as loom's record? Recommended yes (D1): every existing reader works unchanged.
- Support log for darn? Recommended no; split the check (D3).
- `decision add` while Darn-Write runs? Recommended allow (D4); alternative refuse as loom does for Discussion-Write, rejected since the writer spans the whole code phase.
- What outranks the board body? Recommended only `Added after Discussion` entries (D5).

### Turn 4 — Batch: alignment and review wiring

- PR-Gate measure per recipe through fragment stencils (D8).
- `carry_over` on Darn-Bouncer (D6).
- The format pointer: `PATTERN-producer-pointer-rule` needs a contract to point at; at HEAD the format sits inside `loom-template-discussion.md`, so it moves to a spec (D2).

### Turn 5 — Lens review

All six forks returned.

- goal-scope MEDIUM, D5 narrows the entry's "the record … authoritative" only in an assumption: fixed as proposed; D5 now quotes the phrase and says it means the `Added after Discussion` entries.
- goal-scope LOW, no criterion that loom's Discussion-Write is unchanged by D2: fixed as proposed; added that criterion.
- goal-scope LOW, D6's tie to the board sentence: fixed as proposed; D6's rationale names it.
- goal-scope LOW, no criterion for D1's re-spawn: fixed as proposed; added that criterion.
- goal-scope LOW, the parent's procedure (record and message) is implicit: fixed as proposed; D4 has the orch stencil say both.
- goal-scope LOW + leanness MEDIUM (merged), environment note in the record: fixed as proposed; cut from the record, kept in the ledger.
- premises MEDIUM, darn landing order reads unknown in `peers`: fixed differently; the proposed alternative of ranking darn rows in `peers` was not taken, since landing order never changes a verdict and the template handles unknown order; D7 states the behaviour and Scope Out names it.
- premises LOW, the claim that `peers show` works on darn is unverified: fixed differently; the proposal only questioned it, and I dropped the claim from D1's rationale.
- premises LOW, `support_log_path` reads as present on darn: fixed as proposed; D7 has it read `none`.
- premises LOW + scenarios LOW (merged), D4 does not say where the verb gets the recipe: fixed as proposed; D4 names the seed and the failure way forward.
- board-fit MEDIUM, D2 overlaps `twill-fork0`'s edits to the discussion stencil: fixed as proposed; D2 replaces only Step 5's format block, and the overlap is in Open risks.
- board-fit MEDIUM, `run-verbs-shared` will rename the verbs these texts name: fixed as proposed; Scope Out says so.
- board-fit LOW, `orch-split` overlap: fixed as proposed; in Open risks.
- board-fit LOW, sibling arms `twb-d0-sa` and `twb-d0-sf` carry the same entry: fixed differently; the record cannot change that, so it is noted in the ledger.
- board-fit LOW, `shed-tackler` text overlap in `darnwiring.go`: fixed as proposed; in Open risks.
- board-fit LOW, stencil-pinning tests: fixed as proposed; added to Notes.
- scenarios MEDIUM, `decision add` can append to an uncommitted record: fixed as proposed; D4 refuses until the record is committed at HEAD, with a refusal row.
- scenarios MEDIUM, a restarted writer cannot fix a partial record: fixed as proposed; D1 ties "never edit" to a committed record, with a criterion.
- scenarios MEDIUM, Darn-Burler-Fix can edit the record: fixed as proposed; D5 has the rubric forbid it, and the open risk names the fixer.
- scenarios LOW, gate exhaustion and rework scope: fixed as proposed; D3 states the halt and resume, D7 says an agent reject records nothing, and there is a criterion.
- scenarios LOW, `review_max_bounces: 0` leaves the alignment agent as the only judge: fixed as proposed; in D7's bound.
- leanness MEDIUM, assumptions restate rationales: fixed as proposed; each is now one line naming its decision.
- leanness MEDIUM, the PR-Align criterion pins producer behaviour: fixed as proposed; narrowed to darn wiring.
- leanness LOW, D7 history sentence: fixed as proposed; cut.
- leanness LOW, D5 incident sentence: fixed as proposed; cut.
- leanness LOW + pattern-fit LOW (merged), `PATTERN-tolerant-records` line is padding: fixed as proposed; cut.
- leanness LOW, open risk 3 repeats D7's bound: fixed as proposed; cut.
- leanness LOW, doc list stated twice: fixed as proposed; Scope points at Constraints.
- leanness LOW, D1's rationale repeats D4's bound: fixed as proposed; cut from D1.
- leanness LOW, Out bullets carry the rejection reasoning: fixed as proposed; reasoning cut.
- pattern-fit MEDIUM, `refusal-spec.md` row not named: fixed as proposed; D4 names it, plus a row for the uncommitted case.
- pattern-fit LOW, spec and fragments need registering: fixed as proposed; D2 and D8 name `specs.go` and `stencils.go`.
- pattern-fit LOW, `PATTERN-shed-recipe-registry` mis-cited: fixed as proposed; D3 cites `resolveGateSpec`'s doc and error text, and the constraint line is cut.

## Rejected alternatives

- Edit or amend a claimed board entry's body: the operator's Direction picks the decision record instead.
- Turn darn into Discussion-Write, Plan-Write and Webster rows: dropped by the operator as slower and dearer.
- Have darn write a stub support log to satisfy `discussionparser.Validate`: an empty file nobody reads, kept alive only by a check.
- Make the writer's whole record authoritative over the board body: the writer would license its own scope, unreviewed.
- Refuse `decision add` while Darn-Write runs: leaves the incident's gap open.
- One alignment template with darn wording appended to loom's `pr-gate` text: the loom text measures against recorded decisions, which in darn are the writer's unreviewed claims.
- A Go-side check that the record was committed before the first code commit: a cost with no failure it prevents beyond what the gate catches.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, resolved): Record location and format — loom's path and sections (D1).
- Q2 (auto-pick, resolved): Darn support log — none; record-only check (D3).
- Q3 (auto-pick, resolved): `decision add` during Darn-Write — allowed (D4).
- Q4 (auto-pick, resolved): Authority over the board body — `Added after Discussion` entries only (D5).
- Q5 (auto-pick, resolved): Carry-over in darn — yes (D6).
- Q6 (auto-pick, resolved): PR-Gate measure — per-recipe fragment (D8).
- Q7 (auto-pick, resolved): Format pointer — move the format into `contracts/specs/loom-discussion-spec.md` (D2).
- Q8 (auto-pick, resolved): In-flight darn runs — restarted, not migrated (D9).
- Q9 (auto-pick, resolved): `decision add` on an uncommitted record — refused on darn (D4), after the lens review.
- Board note: `twb-d0-sa` and `twb-d0-sf` carry this entry's title and body and run Discussion-Write too; whoever lands one retires the others.
- Environment note: the hub's `_board/_lyx/stencils/` and `_board/_lyx/specs/` carry a build stamp from commit `0850f76e4` ("Darn: decision record for mid-run scope corrections, and PR-Align"), which is not an ancestor of this worktree's HEAD.
  This session read the deployed `loom-template-alignment.md` and `loom-fragment-pr-gate-darn.md` before noticing; the design is drawn from HEAD's code, and nothing was repaired.
