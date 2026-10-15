# Support log — twb-d0-sa

## Interview

### Turn 1 — Where does the Darn review read its scope today?

Autonomous run; answered from the code.
The Darn-Review rubric and the Darn-Burler-Review `fasit` instructions read the board entry live with `lyx board get`; there is no copy.
A claimed entry's body cannot be edited, so no correction can reach the review through the board.
The change therefore touches the rubric, the `fasit` text and the record, not a copy mechanism.

### Turn 2 — When must `decision add` work on a darn run?

The motivating correction arrived while Darn-Write ran.
Auto-pick: the verb works during Darn-Write once the record is committed and unchanged in the working tree (D3); loom's writer-owns-record refusal stays loom's.

### Turn 3 — How does the record get committed mid-session?

`_lyx/discussion/` is committed through `fabricengine` (`commitDiscussion`), never the writer's `git commit`.
Auto-pick: a clean `validate-discussion` on a darn run at Darn-Write commits the discussion pathspec (D2), bounded to that recipe, row and pathspec.

### Turn 4 — How are parent entries protected from the writer?

The writer and the parent can both touch the record during Darn-Write.
Auto-pick: a kept-entries check against the record's whole commit history at the gate, `validate-discussion` and `decision add` (D4).

### Turn 5 — What is authoritative for the review and the alignment agent?

The writer's sections are unreviewed.
Auto-pick: the board entry as amended by `Added after Discussion` entries; the writer's sections are a claim (D5, D8).

### Turn 6 — Did darn leave PR-Align out on purpose?

`a40c7baef`, which added PR-Align, says only "darn's recipe is unchanged", with no reason.
The alignment agent reads the decision record and a plan; darn lacked the record and has no plan.
Auto-pick: add the row, render the plan directory `none`, reuse loom.yaml's alignment keys (D7).

### Turn 7 — Board fit

`run-verbs-shared` keeps the run verbs under `lyx loom` for now, so `decision add` stays there.
`shed-tackler` and `orch-split` touch neighbouring code and the orch stencil; recorded as open risks, not dependencies.

Lens review (six forks, all returned):

- premises, MEDIUM, D10 halt claim false (carry-over gives a bare not-exist error; PR-Align spawns with a missing record): fixed, D10 now gives PR-Align a missing-record note and drops the claim.
- premises, MEDIUM, D7 assumed the prompt fill renders `none` for the plan directory: fixed, D7 names the new fill.
- premises, MEDIUM, D2/D3 verbs read no recipe today: fixed, both decisions name the recipe read.
- premises and pattern-fit, LOW, `PATTERN-driver-choice-single-site` misapplied: fixed, dropped from Constraints.
- premises, LOW, D8 loom wording names `Merge-Review`: fixed, D8 says loom's text is kept verbatim.
- premises, LOW, `commitDiscussion`'s message carries a `loom:` label on darn: rejected, cosmetic.
- pattern-fit, MEDIUM, `PATTERN-gate-self-check-parity` unnamed for D4: fixed in D4 and Constraints.
- pattern-fit, MEDIUM, D2 departs from `PATTERN-fabric-git`'s boundary clause unnamed: fixed, D2 names the clause and why.
- pattern-fit, MEDIUM, D4 way forward needs a weft git read: fixed, the refusal prints the dropped text.
- pattern-fit, LOW, refusal-spec rows implicit: fixed in Constraints, including D2's commit failure.
- pattern-fit, LOW, authority rule stated in three stencils: fixed in part, the `fasit` points at the rubric; the alignment fragment serves another consumer and keeps its own text.
- pattern-fit, LOW, carry-over race on darn: fixed, D3 names the existing transient row.
- scenarios, MEDIUM, D4 history has no lower bound and would inherit another task's entries: fixed, bounded at the merge base, with a scenario.
- scenarios, MEDIUM, D3 differs-refusal way forward unusable past Darn-Write: fixed, row-aware way forward.
- scenarios, MEDIUM, concurrent `decision add`: fixed, D3 serializes under a lock.
- scenarios, MEDIUM, D1 rewrite of an invalid record drops entries: fixed, a rewrite keeps them.
- scenarios, LOW, D2 halted at Darn-Write: fixed, any state counts.
- scenarios and goal-scope, MEDIUM, missing must-cover scenarios (overwrite race, incident end to end, alignment reject on darn, `review_max_bounces: 0`, post-commit confirmation): fixed in Acceptance criteria; the D8 escalate path is covered by the alignment prompt criterion and stays a stencil rule.
- goal-scope, MEDIUM, D5 narrows the entry's authority direction unnamed: fixed, D5 says so.
- goal-scope, LOW, D3 later than "once the record exists": fixed, named in Auto-mode assumptions.
- goal-scope, LOW, Describe change unlicensed: fixed, dropped and listed under Out.
- board-fit, MEDIUM, `shed-tackler` overlap misstated; the real interaction is an arbiter on darn: fixed in Open risks and D3's bound.
- board-fit, LOW, `loom-model-key-names` renames keys D7 reads: fixed, an open risk.
- board-fit, LOW, `run-verbs-shared` builds on this task: fixed, an open risk.
- board-fit, LOW, `orch-split` also moves PR-Gate: fixed, an open risk.
- board-fit, LOW, `discussion-premise-cites` would make the darn record dearer: rejected, speculative and not a conflict.
- board-fit, LOW, duplicate entries `twb-d0-sc` and `twb-d0-sf`: rejected, they are bench arms of this entry.
- leanness, LOW, Notes bullet repeated D5: fixed, it keeps only the pointer.
- leanness, LOW, D7 commit history: fixed, moved to Turn 6.
- leanness, LOW, D10 rationale: fixed, trimmed.
- leanness, LOW, Auto-mode assumptions repeated decisions: fixed.
- leanness, LOW, Scope `In:` restates decisions: rejected, Scope is the boundary list the spec asks for, kept terse.
- leanness, MEDIUM, D3 bound duplicated in Open risks: fixed, the open risk is gone.
- leanness, LOW, stencil pins are plan-writer guidance: fixed, moved to Notes.
- leanness, LOW, ledger repeats turns: rejected, the spec requires self-picks in the ledger.

## Rejected alternatives

- Let the parent edit or amend a claimed board entry's body: the board entry offers it, but it needs a second scope source every reader must merge, while loom already has the record and `decision add`.
- Split darn into Discussion-Write, Plan-Write and Webster rows: the board entry drops it as slower and dearer.
- A new verb for the writer to commit its record: one more surface than a bounded side effect of the validation the writer already runs.
- The writer commits the record with its own `git commit`: `_lyx/discussion/` goes through `fabricengine` (`PATTERN-fabric-git`).
- Let `decision add` append to an uncommitted record: it would commit a half-written writer record and race the writer.
- Compare kept entries against HEAD only: a commit from outside Go could erase an entry from HEAD and pass.
- Make the writer's own record sections authoritative over the board body: they are unreviewed, so an incorrect reading would outrank the operator's entry.
- Darn-specific alignment keys in darn.yaml: YAGNI; `alignment_pr_approve` is a hub-wide landing policy already read from loom.yaml.
- Migrate in-flight darn runs: the operator's norm is restart.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Where does the Darn review read scope? The live board entry (Turn 1).
- Q2 (resolved, auto-pick): `decision add` during Darn-Write? Yes, once the record is committed and unchanged (D3).
- Q3 (resolved, auto-pick): How is the record committed mid-session? A clean `validate-discussion` on a darn run at Darn-Write (D2).
- Q4 (resolved, auto-pick): How are parent entries protected? Kept-entries check against the commit history (D4).
- Q5 (resolved, auto-pick): What is authoritative? Board entry plus `Added after Discussion` entries; writer sections are claims (D5, D8).
- Q6 (resolved, auto-pick): PR-Align for darn? Yes, plan directory `none`, loom.yaml alignment keys (D7).
- Q7 (resolved, auto-pick): In-flight runs? Restart, with a PR-Align missing-record note (D10).
- Q8 (resolved, auto-pick, from the lens review): Concurrent `decision add`? Serialized under a lock (D3).
- Q9 (resolved, auto-pick, from the lens review): Does Describe read the record? No; the board entry does not license it.
- Q10 (resolved, auto-pick, from the lens review): Kept-entries history bound? Commits since the task branch's merge base with its parent (D4).
- Note: `git log` shows a commit `0850f76e4` titled like this task on `main` and other branches, not an ancestor of this worktree's HEAD; its commit message was seen during exploration before that was clear, its code was not read, and the design above is checked against HEAD's code only.
