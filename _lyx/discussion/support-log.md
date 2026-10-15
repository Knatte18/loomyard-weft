# Support log — twb-d0-sf

## Interview

### Turn 1 — task and board

The session ran autonomously with no operator and no advisors.
Read the board entry `twb-d0-sf` and every entry touching it: `run-verbs-shared` (verbs stay under `lyx loom` for now; it names commit `0850f76e`), `shed-tackler` (driver and tackler move into shed, darn included) and `orch-split` (restructures the orch stencil).
`0850f76e` is not an ancestor of HEAD, so it is a later commit on another ref and was not used as a code source.
The sibling entries `twb-d0-sa` and `twb-d0-sc` carry this task's title and brief and run in parallel; the design was settled from HEAD alone.

### Turn 2 — where darn reads its scope today

The Darn-Review rubric and Darn-Burler-Review's `fasit` read the board body through `lyx board get`; darn's Describe stencil does the same.
Darn-Bouncer's recipe comment says darn has no record to carry findings into.
`lyx loom decision add` fails on a darn worktree with a missing-file error and a way forward saying a darn run never has a record; its only writer-ownership refusal names Discussion-Write.
So the change touches the darn writer stencil, the darn recipe, the review rubric and `fasit`, Describe, the decision verb's text and the orch stencil.

### Turn 3 — the record format

The record format lives only in the Discussion stencil's Step 5 at HEAD; `contracts/specs/` holds no discussion spec.
`PATTERN-producer-pointer-rule` rules out copying it into the darn stencil, so it moves into a registered spec both stencils point at (D1).

### Turn 4 — who commits the record

`_lyx/discussion/` is on the records side, which Go commits (Darn-Write's after-session commit already commits the landing directory through `fabricengine`).
`decision add` already commits the record on append, so it needs no committed writer record first (D3, D4).
A parent entry could still be dropped by a later writer or fixer edit, so a history check guards it (D5).

### Turn 5 — PR-Align

Commit `a40c7baef` added PR-Align to loom only and says darn's recipe is unchanged, with no stated reason.
What the alignment agent reads that darn lacked is the decision record; the plan directory renders `none`, which the stencil already reads as absent.
The stencil's `pr-gate` judgment names `Merge-Review`, which darn lacks, so its text goes recipe-neutral (D8, D9).
Darn-Bouncer gains a carry-over so open review findings reach alignment (D7).

### Turn 6 — record written by fork0, lens review

fork0 wrote both files from the settled questions; it moved the record commit from the writer to Go (D3), since `_lyx/discussion/` is on the records side, and dropped the settled `discussion` gate on Darn-Burler-Fix, restored below.
Six lens forks reviewed the record; every one returned a report.

- goal-scope F1, MEDIUM: fixed as proposed — D2 has the writer read the `Added after Discussion` entries as winning scope on every spawn, with an acceptance criterion.
- goal-scope F2, MEDIUM: fixed as proposed — D10 has the orch also message the live Darn-Write session whenever Darn-Write is the running row.
- goal-scope F3, LOW, merged with scenarios 5, MEDIUM: fixed as proposed — Scope names that D5 covers loom records, D5's bound states the loom effect, and an acceptance criterion covers a loom record.
- goal-scope F4, LOW: fixed as proposed — D6's rationale says it departs from the entry's wording.
- premises 1, MEDIUM: fixed as proposed — D8 reuses the shared `landingAlignmentDeps` and changes only the plan-dir rendering, which the recipe decides.
- premises 2, MEDIUM, merged with premises 3, MEDIUM: fixed as proposed — D2 fixes an invalid record in place, keeping every parent entry and carry-over block, and its bound says carry-over blocks are kept by the stencil rule, not a mechanical check.
- premises 4, LOW: fixed as proposed — D3 names the parent-review directories in the commit pathspec, absent on darn.
- premises 5, LOW: fixed as proposed — the finding confirms D8's reject path and proposes no change; no edit.
- premises 6, LOW, merged with scenarios 4, MEDIUM, and scenarios 8, LOW: fixed as proposed — D4's bound states the unlocked race, the restore window and the sweep of an unfinished writer record; the marker alternative scenarios 4 offered is in Rejected alternatives.
- leanness 1, LOW: fixed as proposed — D8's rationale drops the commit history, which Turn 5 holds.
- leanness 2, LOW: fixed as proposed — D1, D3 and D5 rationales no longer repeat the PATTERN entries Constraints lists.
- leanness 3, LOW: fixed as proposed — Auto-mode assumptions is one line pointing at D6 and D8; the `0850f76e` note stays in Turn 1.
- leanness 4, LOW: fixed as proposed — the `shed-tackler` bullet is folded into one merge-risk bullet.
- leanness 5, LOW: fixed as proposed — the `PATTERN-tolerant-records` constraint is cut.
- leanness 6, LOW: fixed as proposed — the first Notes bullet is cut.
- leanness 7, LOW: fixed as proposed — the last acceptance criterion names prompt behaviour, not stencil wording.
- board-fit MEDIUM (`twill-fork0`): fixed as proposed — the merge-risk bullet names it.
- board-fit LOW (sibling runs and `0850f76e`): fixed differently — the proposed line in Auto-mode assumptions was not added, since leanness 3 cut that section to the auto-picks and the plan reads HEAD; Turn 1 records the sibling runs instead.
- board-fit LOW (`discussion-premise-cites`): fixed as proposed — named in the merge-risk bullet.
- board-fit LOW (`shed-tackler` hand-off): fixed as proposed — the finding is informational and the merge-risk bullet covers it; no further edit.
- pattern-fit BLOCKING: fixed as proposed — D5 makes `validate-discussion` a third caller of one shared function and names the `PATTERN-gate-self-check-parity` Pairs change; an acceptance criterion covers `validate-discussion`.
- pattern-fit MEDIUM, merged with scenarios 3, MEDIUM: fixed differently — the proposed way forward through the committed copy, or lyx restoring the entry, was not taken, since reading the copy needs records-side git and an automatic restore hides the drop; the finding instead quotes the entry from its last committed version, so the session restores it with Edit, and D2 allows that one edit.
- pattern-fit LOW (told seam): fixed as proposed — D5 hands the history reader to the gate as a seam built in `loomcli` wiring.
- pattern-fit LOW (config docs): fixed as proposed — D8 and Constraints name the alignment keys' doc comment in `internal/loomengine/config.go`.
- scenarios 1, MEDIUM: fixed as proposed — D2 puts the `discussion` gate on Darn-Burler-Fix as well.
- scenarios 2, MEDIUM: fixed as proposed — D5's reference is the record's commits on the run's branch since its merge base with the parent, with an acceptance criterion.
- scenarios 6, LOW: fixed as proposed — D11 states the halt and its way forward, with an acceptance criterion.
- scenarios 7, LOW: fixed as proposed — acceptance criteria cover the resume budget, the rework spawn with parent entries and `decision add` on a half-written record.

## Rejected alternatives

- **Let the parent edit or amend a claimed board entry's body.** It changes the board's claim model and every reader of the body for one correction channel, while the record already has an append verb, a commit and a history; the board entry's own direction picks the record.
- **Split darn into Discussion-Write, Plan-Write and Webster rows.** Dropped by the operator: each row would explore the code anew, slower and dearer.
- **Hold the whole darn record authoritative over the board body.** The record is unreviewed before coding, so the writer would license its own scope; only the parent's `Added after Discussion` entries win (D6).
- **Refuse `decision add` while Darn-Write runs, as for Discussion-Write.** That leaves the incident's window, a correction during the writer's session, without a record.
- **Refuse `decision add` until a marker records the writer's first clean validation.** It closes a narrow write race at the cost of a new marker record, and reopens the incident's window for every correction sent before the writer validates; D4 states the race as a bound instead.
- **Restrict the history check to darn.** It would leave a loom record's parent entries droppable by a redesign, and add a recipe branch to the gate.
- **Have the writer commit the record with git.** The discussion directory is on the records side, which the writer's code commits do not reach, and `PATTERN-fabric-git` keeps that git in `fabricengine`.
- **Check only HEAD's committed record for dropped entries.** A commit made after the drop would launder it; the whole history is the reference.
- **Darn-only history check.** A recipe branch in the gate buys nothing; loom's record follows the same append-only rule.
- **A told per-recipe fragment for the alignment `pr-gate` judgment.** One neutral text covers both recipes, since the plan path already reads `none` when absent.
- **Alignment keys in darn.yaml.** A second copy of a hub-wide alignment setting nobody asked for; CLAUDE.md already treats loom.yaml's `alignment_pr_approve` as the switch for agent landings.
- **Migrate in-flight darn runs.** The darn recipe restarts runs across row changes already.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (resolved, auto-pick): Where does the record format live for both writers? — A new registered spec, `contracts/specs/loom-discussion-spec.md`, both stencils pointing at it (D1).
- Q2 (resolved, auto-pick): When does Darn-Write write the record, and may it edit it later? — Before any code, validated clean, never edited after; a later spawn rewrites it only when absent or invalid (D2).
- Q3 (resolved, auto-pick): Who commits the record? — Go: Darn-Write's after-session commit and `decision add`'s own commit (D3).
- Q4 (resolved, auto-pick): Is `decision add` refused while Darn-Write runs? — No; bounded by caller checks, append-only entries, the immediate commit and the history check (D4).
- Q5 (resolved, auto-pick): How is a parent entry protected from a later edit? — A history check on `Added after Discussion` headings in the discussion gate and `decision add`, all recipes (D5).
- Q6 (resolved, auto-pick): What does the Darn review hold authoritative? — The board entry as amended by the record's parent-written entries; the writer's sections are a claim (D6).
- Q7 (resolved, auto-pick): Did darn leave PR-Align out on purpose? — No stated reason in `a40c7baef`; the missing input was the record (D8).
- Q8 (resolved, auto-pick): Which config governs darn's PR-Align? — loom.yaml's alignment keys (D8).
- Q9 (resolved, auto-pick): How does the alignment stencil serve a darn PR-Gate? — Recipe-neutral `pr-gate` text, plan read only when not `none` (D9).
- Q10 (resolved, auto-pick): Do darn's open review findings reach alignment? — Yes, through `carry_over: Darn-Review` on Darn-Bouncer (D7).
- Q11 (resolved, auto-pick): In-flight darn runs? — Restarted, not migrated (D11).
- Q12 (resolved, auto-pick): Loom's gap before Discussion-Write has written the record? — Out of scope: the board entry notes it and asks nothing of it.
- Q13 (resolved, auto-pick): Which commits does the history check hold a record to? — The record's commits on the run's branch since its merge base with the parent (D5).
- Q14 (resolved, auto-pick): Which rows carry the `discussion` gate on darn? — Darn-Write and Darn-Burler-Fix (D2).
