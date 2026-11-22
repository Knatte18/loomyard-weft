# Support log — move-task-by-handback

## Interview

### Turn 1 — exploration (autonomous mode)

No operator answers questions in this session; every pick below is an auto-pick.
Board read: no `Decision for move-task-by-handback:` note exists.
Touching entries: `stale-records-ref-blocks-rerun` (a re-run after `cancel --remove` is blocked by a stale tracking ref) and `orch-autonomy-mode` (text overlap in `contracts/orchrole/`).
Three exploration subagents read cancel, run records and the orch's role text.
Facts settled:

- `lym batten cancel` sets the entry back to `draft`, which frees the run lock; only `--remove` makes archive tags, and its output lists them as `archive_tags`.
- Cancel without `--remove` blocks re-running the slug.
- `--remove` keeps uncommitted pair files under a local snapshot tag.
- No verb reads an archived run; `fabricengine.SlugRecordsTags` serves only re-adoption.
- "Hand-back" already names the shed driver's stop report and a tackler verdict.
- The orch role text names cancel but no move procedure.

### Turn 2 — design picks

- Verb or guidance: guidance alone (D1).
- Note name and place: `_lym/move-note.md`, term `move note` (D2, D8).
- No live writer: the orch writes the note (D3).
- Same entry or new task: the same entry (D5).
- Commits: read only (D6).

Lens review, one line per finding, duplicates merged:

- bound-breaker 1, existing-paths 1, interactions 5 (a body edit never reaches a discussed entry's run): fixed, D4 step 3 writes into the stored record through `--record-file`, with a constraint.
- interactions 1, bound-breaker 3 and 6 (the run moves on while the note is written): fixed, D2 pauses the run with `lym batten pause` before the ask.
- interactions 2 (a live run with no writer session falls through): fixed, D3's trigger is now "none of D2's writers is live".
- interactions 3 (a partial cancel lists no tags): fixed, D4 step 2 re-runs cancel until it completes.
- interactions 4, walkthrough 1 (`archive_tags` holds only the code tag; the records tag is unreported): fixed, Constraints corrected and the carrier names the records tag pattern.
- walkthrough 2 (empty `archive_tags`): fixed, the carrier names no tag.
- walkthrough 3 (an early-row move has no commits): fixed, one sentence in D6.
- bound-breaker 2, existing-paths 2 (the snapshot recovery claim is unpinned): fixed, the claim is dropped and the orch copies the note to its `.scratch/` before the cancel.
- board-scope 1 (the note is not in the archive tag, against the entry): fixed, D2 states the departure and its reason.
- bound-breaker 4 and 5 (owner machine across parents): fixed, a constraint states the owner follows the parent and the fresh run needs nothing local; `owners.go` reads a recorded owner only on a beam project, so the task's `owner_machine` needs no reset.
- existing-paths 3 (old archive and snapshot tags meet a same-slug re-run): rejected, a cancel with `--remove` then a re-run of the slug is the flow the orch role already prescribes, so those paths are not new to this design.
- existing-paths 4 and 5 (project change and release survive): no change needed.
- leanness 1 (D4 rationale repeats Constraints): fixed, cut to a pointer.
- leanness 2 (Goal's last sentence): fixed, cut.
- leanness 3 (Scope bullet on the task's own entry): fixed, cut.
- leanness 4 (plan-writer notes): fixed, section removed.
- leanness 5 (`PATTERN-github-auth` constraint): fixed, cut.
- leanness 6 (nickname "Merriam" in the record): fixed, "the Webster master session".
- leanness 7 (support log restates facts): rejected, the support log is the deliberation record and the restatement is short.
- board-scope 2 (term renamed from the entry's): no change, D8 states the rename.

## Rejected alternatives

- A `lym batten move` verb wrapping cancel and upsert: it cannot drive the writer's note, and the rest is two existing verbs; rare use does not pay for a cross-module verb.
- The re-parent verb of the study: two durable stores with no atomic write, a narrow board write past the run lock and an ancestry refusal; the operator dropped it.
- Cherry-picking the old commits onto the fresh run: brings back the ancestry question of the study's section 3.
- A new task under the new parent: loses the entry's handle and history for no gain, since a draft's `project` is freely editable.
- Committing the move note to the records branch: needs a new committing writer under `PATTERN-run-records`; the board body is already a durable copy.
- Calling the file a hand-back: collides with the driver's hand-back.
- Cancel without `--remove`: makes no archive tags and blocks the re-run.
- Asking the writer without holding the run: the tip SHA and merge base can drift before the cancel.
- Refusing a move across owner machines: the owner follows the parent, and the fresh run needs nothing local to the old machine.
- Clearing a discussed entry's stored record instead of adding to it: throws away decisions the fresh run still needs.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Verb or orch guidance? Resolved, auto-pick: guidance alone (D1), since every step but the note is an existing verb.
- Q2 Re-use or read the old commits? Resolved, auto-pick: read only (D6), to avoid the ancestry question.
- Q3 Same entry or new task? Resolved, auto-pick: same entry (D5).
- Q4 What to call the file? Resolved, auto-pick: move note (D8), since hand-back is taken.
- Q5 A run with no live session? Resolved, auto-pick: the orch writes the note (D3).
- Q7 Where does the note go for a discussed entry? Resolved, auto-pick: the stored record's `## Notes for the plan writer` through `--record-file` (D4), from the lens review.
- Q8 Hold the run before the ask? Resolved, auto-pick: `lym batten pause` first (D2), from the lens review.
- Q6 The re-run block of `stale-records-ref-blocks-rerun`? Open, recorded under Open risks.
