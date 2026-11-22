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
- Note name and place: `_lym/move-note.md`, term `move note` (D2, D8); review round 2 moved the note to the orch's `.scratch/`.
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
- Reading the note's tip SHA and merge base without holding the run: they can drift before the cancel.
- Holding the run with `lym batten pause`: the inner loom or Webster run reads only the `lym tender pause` record, so it keeps committing.
- Holding the run with `lym tender pause` before the ask: the pause holds every delivery into the writer's pane, so the ask never arrives.
- Taking the code archive tag from `archive_tags`: it names at most the local tip's tag, created before the teardown child pushes it.
- Refusing a move across owner machines: the owner follows the parent, and the fresh run needs nothing local to the old machine.
- Clearing a discussed entry's stored record instead of adding to it: throws away decisions the fresh run still needs.

## Review rounds

### Round 1

- F1 (batten pause does not hold the inner run; tender pause holds the ask): fixed, D2 asks first, then holds with `lym tender pause`, and the orch reads the tip after the hold; D7 lifts with `lym tender resume`.
- F2 (cancel returns before its tags reach origin): fixed, Constraints rewritten, the code tag is named from the recorded tip and confirmed by `ls-remote`.
  This corrects Turn 1's fact that `archive_tags` lists the archive tags and drops Turn 2's empty-`archive_tags` rule.
- F3 (no check that the new parent takes a member): fixed, D4 step 2 checks before the cancel.
- F4 (stale-ref workaround only in Open risks): fixed, D4 step 5 runs the pruning fetch.
- F5 (partial finish and refusal not told apart): fixed, D4 step 3 defines both by cancel's output.
- F6 (H2 move note in a stored record): fixed, an H3 under `## Notes for the plan writer`.
- F7 (glossary names the writer): fixed, as proposed.

### Round 2

- F8 (the writer cannot write `_lym/move-note.md`): fixed, D2 asks for the first three parts in the reply and the orch writes the note in its `.scratch/`; D3 takes a refusing writer as no live writer.
  This replaces Turn 2's note place.
- F9 (refusals filed as partial finishes): fixed, the two cases are told apart by the way forward each names, since the output does not show whether the cancel ended anything; no slugless `lym tender resume`.
- Fixer self-check: the ready record is read on the beam branch, the records branch is named `<task branch>-weft`, the manual tag push runs from the prime, the records-branch stop names the note's scratch path and where the move goes on, and D7 covers the owned-elsewhere refusal.
- F10 (teardown wait covers the code tag alone): fixed, D4 step 3 confirms the tag and the records branch's absence, bounded at ten minutes with a manual tag push; D6 fetches the tag.
- F11 (ready flag is no board field): fixed, D4 step 2 reads `beam-ready.json`; a refused `project` write keeps the note under the old parent.

### Round 3

- F12 (ready record read on the code branch): fixed, D4 step 2 reads `beam-ready.json` on the beam's records branch `<beam branch>-weft`,
  since `lym beam ready` commits it in the run-records worktree with no push;
  for a beam on another machine an absent file on origin proves nothing, so the orch asks that beam's orch or the operator whether `lym beam ready` ran.
  This replaces Round 2's read on the beam branch.

## Question ledger

- Q1 Verb or orch guidance? Resolved, auto-pick: guidance alone (D1), since every step but the note is an existing verb.
- Q2 Re-use or read the old commits? Resolved, auto-pick: read only (D6), to avoid the ancestry question.
- Q3 Same entry or new task? Resolved, auto-pick: same entry (D5).
- Q4 What to call the file? Resolved, auto-pick: move note (D8), since hand-back is taken.
- Q5 A run with no live session? Resolved, auto-pick: the orch writes the note (D3).
- Q7 Where does the note go for a discussed entry? Resolved, auto-pick: the stored record's `## Notes for the plan writer` through `--record-file` (D4), from the lens review.
- Q8 Hold the run before the ask? Resolved in review round 1: ask first, then hold with `lym tender pause` (D2), since that pause holds the ask's delivery and `lym batten pause` holds no inner run.
- Q6 The re-run block of `stale-records-ref-blocks-rerun`? Resolved in review round 1: D4 step 5 runs the pruning fetch until that entry lands.
