# Support log — claimed-scope-edit

## Interview

The session ran autonomously: no operator answered, so every call below is an auto-pick recorded in the Question ledger.

1. Read the board entry and the board.
   Touching entries: `twill-spawn-after` (running darn task editing the twill directive and Discussion stencil's fork procedure), `glyph-use-audit` (the source of the `webster-pack-last` scope), `orch-split` (restructures the orch stencil, which already tells the orch to use `decision add`), `lighter-test-rules` (stencil-wording pins).
2. Explored at module level: `contracts/recipes/darn-recipe.yaml`, the PR-Align row and comments in `loom-recipe.yaml`, `internal/loomcli/decision.go`, `internal/discussionparser` (Validate requires the support log and seven sections), `contracts/stencils/darn/darn-template-write.md`, `loom-rubric-darn-review.md`, `landing-template-describe-darn.md`, `loom-template-alignment.md`, `internal/alignment/doc.go`, `internal/landingshed/pralign.go`, `landingAlignmentDeps`, and commit `a40c7baef`.
3. Answered the entry's first check: the Darn review reads its scope live from the board body, through `lyx board get`, named in both the rubric and the review row's `fasit` instructions; there is no copy.
4. Answered the entry's second check: `a40c7baef` says only "darn's recipe is unchanged"; PR-Align's deps need a decision record and snapshot, and its `pr-gate` judge reads recorded decisions and carried-over open risks, which darn lacked. Nothing else ties PR-Align to loom.
5. Settled D1–D10 as auto-picks; spawned six lens forks before writing.
6. Lens review, all six forks returned; duplicates merged:
   - board-fit + goal-scope BLOCKING, orch stencil contradicts D5/D6/D9 and is out of Scope: fixed, D11 and a Scope line added.
   - board-fit + goal-scope MEDIUM, `orch-split` overlap unnamed: fixed, Open risks line.
   - board-fit LOW, `twill-spawn-after` overlap wider: rejected, the existing Open risk covers it and the landing order needs no record change.
   - board-fit + leanness LOW, `CLAUDE.md` edit may change nothing: fixed, Constraint reworded to "stays recipe-neutral".
   - board-fit LOW, `discussion-premise-cites` would also gate darn's record: rejected, no conflict today; that entry decides its own reach.
   - scenarios BLOCKING, the writer's own record would grade its own diff: fixed, D7 now lets only `Added after Discussion` entries override the board body and treats the writer's sections as a claim; the narrowing of the entry's wording is stated in D7.
   - scenarios BLOCKING + premises MEDIUM + pattern-fit MEDIUM, carry-over halts a record-less in-flight run: fixed, A4 now restarts in-flight darn runs (the recipe's own policy), the record-less fallbacks in D7 and D9 are dropped, and the halt is an Open risk.
   - pattern-fit MEDIUM + premises LOW, no sanctioned path for the writer to commit `_lyx/discussion/`: fixed, D4 has Go commit both files at the gate's pass; D3 says the writer commits neither.
   - scenarios MEDIUM + pattern-fit LOW, `decision add` racing the writer's commit or a half-written support log: fixed, D5 drops the uncommitted-record check and relies on the verb's existing post-append check and commit-failure refusal, both stated.
   - scenarios MEDIUM, partial record after a writer death: fixed, D3 rewrites the files when absent or invalid; acceptance line added.
   - scenarios MEDIUM, correction the parent never records: fixed, D6 names it in the change description; acceptance line added.
   - scenarios MEDIUM, missing acceptance scenarios: fixed for respawn, writer death, snapshot escalation and invalid append; the halted-Darn-Write case is covered by the post-append check.
   - scenarios LOW, in-flight resume policy contradicts the recipe header: fixed, Constraints now say restarted, not migrated.
   - scenarios LOW, alignment reject on the writer's own scope reading: fixed, Open risks line.
   - goal-scope LOW, pre-writer gap in D5's way forward: fixed, the way forward names the board-note fallback when no writer is live.
   - goal-scope LOW, D2 and D8 past the literal asks: rejected, both justified as the fork itself says.
   - goal-scope LOW, D9's no-record branch recipe-neutral: moot, branch dropped with A4.
   - premises LOW, `plan_dir` needs `none` rendering: fixed, D9 says the prompt renders an empty plan directory as `none`.
   - premises LOW, D9's no-record check is new: moot, branch dropped.
   - pattern-fit LOW, specs marker and registration: fixed in D2.
   - pattern-fit LOW, exhausted-gate refusal row for Darn-Write: fixed in Constraints.
   - leanness LOW: Goal second paragraph cut, D10 second clause cut, D9 rationale shortened, first Notes bullet cut, speculative Open risk cut, D1 consumer list generalized; all fixed.

## Rejected alternatives

- Edit a claimed board entry's body, or append a recorded amendment to it: the operator's direction chose the record; it would also need every reader to re-read the body and a who/when trail the record already has.
- Turn darn into a loom without perches: dropped by the operator for cost and speed.
- A darn-only validator that checks the record without a support log: a second validator and a recipe-aware append path for one saved file.
- Point the darn stencil at the Discussion stencil's Step 5 instead of extracting a spec: it hands the darn writer Discussion's interview and fork procedure, and a template with markers is not a format contract.
- Let the darn writer keep editing its record through the run: races the parent's `decision add` appends.
- Refuse `decision add` for the whole of Darn-Write, as for Discussion-Write: the board entry's failure happened during Darn-Write, so the correction would again live only in a message.
- Give darn its own `darn.yaml` alignment keys: two switches for one landing route, and darn already reads loom.yaml for its perch models.
- A record-less fallback for in-flight darn runs (board-only review, PR-Align deferring to the parent): new branches for a handful of runs the recipe's policy already restarts.
- The darn writer commits its own record: no stencil gives an agent a sanctioned path to commit `_lyx`, and loom has Go commit it.
- Refuse `decision add` during Darn-Write until the record is committed: the existing post-append check already refuses a partial record, and the extra check needed a git read.
- Make the whole darn record authoritative over the board body, as the entry worded it: the writer's own unreviewed reading would become the review's answer key.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Where does the Darn review read scope today? Resolved by reading: the live board body. Sets the change to the rubric and the `fasit` text.
- Q2 (auto-pick): Did darn omit PR-Align on purpose? Resolved as A1: only for lack of a record.
- Q3 (auto-pick): Darn support log required or dropped? Required (D1, A3).
- Q4 (auto-pick): Where does the record format live? A new spec (D2).
- Q5 (auto-pick): Is the record enforced? A `discussion` gate on Darn-Write (D4).
- Q6 (auto-pick): May `decision add` run during Darn-Write? Yes, on a committed record (D5).
- Q7 (auto-pick): Carry-over for Darn-Review? Yes (D8), so PR-Align sees the perch's open findings.
- Q8 (auto-pick): Which config governs darn's PR-Align? loom.yaml's alignment keys (D9, A2).
- Q9 (auto-pick, revised after the lens review): In-flight darn runs with no record? Restarted, not migrated (A4).
- Q10 (auto-pick): Who commits the darn record? Go, at the `discussion` gate's pass (D4).
- Q11 (auto-pick): Which parts of the darn record override the board body? Only `Added after Discussion` entries (D7).
- Q12 (auto-pick): Does the orch stencil change? Yes (D11).
