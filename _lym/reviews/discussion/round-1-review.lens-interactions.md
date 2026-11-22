```yaml
lens: interactions
outcome: reported
```

Task: interactions lens, checking each pair of the record's decisions that act on the same state.

Pairs checked:
- D2 pause × D4 step 2 cancel: held. Cancel sets the pause aside (`cancelcmd.go:462`) and lifts the slug's own pause entries (`cancel.go:572`).
- D2 tip SHA and merge base × D4 step 2 archive tag × D6 diff: **broken**. The pause does not hold the inner run, so the note's merge base and tip can drift from what the tag holds.
- D2 note at `_lym/move-note.md` × D4 step 2 `--remove`: held. The orch copies the note to `.scratch/` before the cancel.
- D2 × D3 (writer list against trigger): held. D3's trigger is "none of D2's writers is live".
- D4 step 2 `archive_tags` × D4 step 3 carrier × D6 "empty diff": **broken**. An empty list does not mean there were no commits.
- D4 step 2 "re-run until it completes" × D7 "when cancel refuses, stop and lift the pause": **broken**. The record never says how the orch tells a partial finish from a refusal.
- D4 step 3 body edit × D1 bound (draft clears the run lock): held for the body path (`runlock.go:167`). The record-file path bypasses the lock anyway (`decision.go:45`), so the bound is harmless.
- D4 step 3 × D5 (same entry, `project` change): held. A draft entry takes scope edits.
- D4 step 4 release × D5: held.
- D5 re-run × Open risks (`stale-records-ref-blocks-rerun`): **broken**. No decision or acceptance criterion puts the workaround into the procedure.
- D7 × D2 for work that reached its parent: held. The orch-role text says a batten pause does not hold that run (`orch-role.md:62`), and D7 sends it to finish its landing.
- D8 glossary line × D3: **broken (wording)**. The line says the writer learned it, but under D3 the orch writes the note.
- Constraint "archive tags are on origin" × D4 step 2: broken; same root as the `archive_tags` finding.
- Acceptance criteria × D1 to D8: each criterion matches its decision's text. The D4 criterion repeats the `archive_tags` gap and the D2 criterion repeats the pause premise.

Findings:

1. **BLOCKING · design · `decision-record.md` D2 with D4 step 2 and D6.**
   D2 claims the pause keeps the note's tip and merge base equal to what the cancel archives, but `lym batten pause` stops only batten's outer run.
   - Evidence: `internal/battenshed/innerrun.go:782-783` and `:400-401`. Batten's Run-Shed reads `pause_requested` and pauses at batten's own row boundary.
   - The inner loom and webster run is held only by the tender pause record (`internal/loomcli/pausehold.go`, `internal/webstercli/batchpause.go:48-54`).
   - So a Webster batch commit or a Parent-Align merge-in can land after the note is written. D6's "diff from the note's merge base to the tag" then misses commits or pulls in old-parent commits.
   - The tender pause that would hold the run also holds creel delivery to the writer's pane (`internal/creelengine/doc.go:258`). D2's two steps cannot both hold with either lever.

2. **BLOCKING · design · `decision-record.md` D4 step 2 with D6 and the Constraints.**
   `archive_tags` does not reliably name the tag D6 reads.
   - In production `DiscardTaskBranch` hands the remote half to a detached child (`remoteteardown.go:274-286`, `childHold.finish` at `:319-333`). It returns only the local tip's tag, created but not yet pushed (`taskbranchdiscard.go:79-89`, `:93-97`).
   - With no local branch, the list is empty even though the child archives origin's tip (`taskbranchdiscard.go:68-78`, `archiveDiscardedTips` at `:109-111`).
   - So D4's "empty list → the carrier names no tag" drops a real tag. D6's line that an empty diff means no code commit misreads that case.
   - A failed push is only mailed as a notice (`remoteteardown.go:184-189`). The Constraints line "pushes the tags", and the one that the fresh run needs nothing local because the tags are on origin, do not hold when cancel returns.

3. **MEDIUM · consistency · `decision-record.md` D4 step 2 with D7.**
   D4 has the orch re-run a cancel that reports a partial finish; D7 has it lift the pause and stop when cancel refuses.
   - The cancel verb reports both the same way: an error envelope through `fail` (`cancelcmd.go:89-92`, `:122-125`).
   - The run-phase errors that follow `WriteIntent` and `EndForCancel` come after processes have ended (`cancel.go:372-409`, `clearRunPhase` at `:529-566`).
   - If the orch reads one of those as a refusal and runs `lym batten run`, it tries to resume a half-cancelled run.
   - Neither decision says which output field separates the two cases. `end_only` is the only marked partial finish.

4. **MEDIUM · design · `decision-record.md` D5 with Open risks.**
   D5's same-slug re-run depends on the pruning fetch, but no decision or acceptance criterion puts that step into the procedure.
   - The procedure's text in `role-beams.md` would leave the orch to hit dobby's refusal.
   - Evidence: the board entry `stale-records-ref-blocks-rerun`, with its workaround `git -C <hub>/loomyard-weft fetch --prune origin`.

5. **NIT · consistency · `decision-record.md` D8.**
   The glossary line says "what a cancelled run's writer learned", but under D3 the orch writes the note when no writer is live.

Tool calls: none.
