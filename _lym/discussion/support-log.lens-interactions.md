```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked each pair of the record's decisions that act on the same state, then checked every acceptance criterion and constraint against the decisions it touches.

**Findings**

1. **MEDIUM, D2 × D4 (note, then cancel): the run keeps moving while the note is written.**
   - Nothing holds the run between the creel ask and the cancel. The driver can advance rows, Merriam's implementers can commit, and Parent-Align can merge the old parent in.
   - So the note's tip SHA, and possibly its merge base, can fall behind the tip that cancel archives.
   - D6 then reads "the diff from the note's merge base to the tag". If Parent-Align ran in between, that diff includes parent commits that are not the task's work.
   - The row can also end and the writer change before the orch reads the note.
   - Evidence: D2 says the orch asks "the run's live writer". D4 step 1 comes before step 2 with no `lym batten pause` or `lym tender pause` between them. The constraints say a pause exists that keeps sessions alive.
   - Fix: pause the run before the ask, or take the commits section from the archive tag after the cancel rather than from the note.

2. **MEDIUM, D2 × D3 (who writes the note): a live run with no writer session falls through.**
   - D2 lists only the Discussion writer, the plan writer and Merriam as the live writer. D3 applies only "when no session of the run is live".
   - A run sitting in a review, gate or landing row has live sessions (a reviewer, the tackler) but none of D2's writers. Neither decision covers that case.
   - Evidence: D2's list against D3's trigger.
   - Fix: D3's trigger should read "no writer of D2 is live".

3. **LOW, D4 × D7 (a cancel that fails partway): "succeeds" is undefined.**
   - With origin unreachable, cancel only ends processes and strands, and a re-run finishes it. That partial run lists no `archive_tags`.
   - D4 step 2 assumes the tags are in the output. D7 says only "a cancel that succeeds", with no criterion for success and no instruction to re-run until the tags are reported.
   - Evidence: the cancel subagent's point 6 ("a re-run finishes the cancel"), set against D4 step 2.

4. **LOW, D4 × D6 (old decisions): the records tag is dropped.**
   - D4 keeps both tags but writes only the code archive tag into the body.
   - D3 has the orch read the old decision record and plan, but once `--remove` deletes the records branch, those exist only under the records tag. D6's fresh run has no pointer to them.
   - Fix: record both tags in the `## Move note` section.

5. **LOW, D5 × D4 step 3 (body edit on a discussed entry).**
   - An entry that carries `discussed` and `record_body_sha256`, as a motif or Sketch-filed entry does, gets a body change.
   - The record does not say whether the stored record still matches, or whether the fresh run then repeats Discussion.
   - This is mostly in the existing-paths lens's territory; I note it only.

**Pairs checked**
- D1 × D4 (D1's bound relies on cancel setting the entry to `draft` before the upsert): held.
- D2 × D4 (note before cancel, and the note's commit data): broken, finding 1.
- D2 × D3 (writer, or orch when no session is live): broken, finding 2.
- D2's snapshot-tag bound × D4 step 2's `--remove`: held.
- D2 × D8 (the term and the file name): held.
- D4 × D5 (same entry, `project` changed after cancel): held.
- D4 × D6 (code archive tag, then a diff from the merge base): held for the code side. The merge base stays reachable through the tag's history, apart from finding 1.
- D4 × D7 (a refused or partial cancel): partly broken, finding 3.
- D4 step 4 × D5 (release by the new parent's orch, same entry): held.
- D5 × the open risk on `stale-records-ref-blocks-rerun`: held, stated.
- D6 × D3 (the orch reads old records, the fresh run gets no pointer to them): broken, finding 4.
- D7 × D2 (a refused cancel after the writer was interrupted): held. The run just continues.

**Acceptance criteria and constraints**
- Every id D1 to D8 is named in a criterion.
- The criteria for D2 and D4 lock in the gaps of findings 1 to 3 as written; they need updating with the fixes.
- The "No PATTERN entry changes" constraint against D2's uncommitted note: held.
- The constraint that cancel without `--remove` makes no tags, against D4: held.

I made no further tool calls.
