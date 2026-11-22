```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism step by step on one concrete case, passing each step the data the step before it actually produces.

**Decisions walked:**
- D1, the move run as a procedure the orch follows: every step is one of the verbs D2 to D7 name. Purpose met, apart from the problems found under D2 and D4.
- D2, case: a run in Webster, batch 2 running. `lym batten pause` sets `pause_requested` on batten's own status only. Batten's Run-Shed check ends and batten pauses at its own row boundary. The child loom or Webster run keeps going, so batches commit and Parent-Align can merge the parent in. The tip SHA and merge base in the note then drift from what the cancel archives. Purpose not met.
- D3, case: a run halted at Code-Review. The orch writes the note from the record, plan, status and branch log. Purpose met.
- D4 step 2, case: a production cancel with `--remove`. `archive_tags` carries only the local tip's tag, and the push of that tag is handed to a detached teardown child. With no local branch the list is empty, though the child archives origin's tip. Purpose at risk.
- D4 step 3, undiscussed entry: the body gets a `## Move note` section and the run lock is free once the status is `draft`. Purpose met.
- D4 step 3, discussed entry: the stored record is seeded and the Discussion block is skipped. Any H2 headings the note carries become extra H2 sections in a record whose format admits no others. Purpose at risk.
- D4 step 3, moving into a beam whose pawl run holds its entry: the cancel has already run before the membership write is refused. The task is left cancelled under its old parent. Purpose not met.
- D4 step 4, release by the new parent's orch: purpose met.
- D5, same slug run again after `--remove`: purpose met, given the workaround named in Open risks.
- D6, the diff from the note's merge base to the code tag: met when the tag is on origin. A move with no commits gives an empty diff, as the record says.
- D7, cancel refuses because the work reached its parent: `lym batten run` lifts batten's pause, and its way forward finishes the landing. Purpose met.
- D8, the glossary line: met, apart from one wording mismatch with D3.

**Findings:**

1. BLOCKING · design · `decision-record.md` D2 (the pause step).
   - Summary: `lym batten pause` stops only batten's outer run, at a Run-Shed check. The inner loom or Webster run keeps committing and running Parent-Align, so D2's claim that no row, batch commit or Parent-Align merge-in moves the branch does not hold.
   - Evidence: `internal/battenshed/innerrun.go:400-401` and `:782-783` (the pause ends batten's check, then "the engine then pauses at the row boundary"). `internal/battencli/cli.go:379-381`. The inner run honours only `pauserecord` (tender) pauses: `internal/webstercli/batchpause.go:33-54`.
   - Fix: hold the inner run some other way. A `lym tender pause` holds it, but it also holds creel delivery to the writer (`internal/creelengine/doc.go:258`). The design has to settle that conflict and pin the result.

2. MEDIUM · design · `decision-record.md` D4 step 2 and the Constraints archive bullet.
   - Summary: `archive_tags` does not reliably name a tag that is on origin.
     - In production the remote half, which pushes the tags and deletes the branch on origin, is handed to a detached child after the command returns. `archive_tags` then lists only the local tip's tag, pushed later.
     - A failed push only sends a notice to the orch.
     - With no local branch the list is empty, though the child archives origin's tip, so "the carrier names no tag" drops a real tag.
     - Diverged local and origin tips get two tags, and only one is reported.
   - Evidence: `internal/fabricengine/taskbranchdiscard.go:30-37`, `:79-97`; `internal/fabricengine/remoteteardown.go:269-286`, `:319-332`, `:184-189`.
   - Fix: wait for the remote-teardown outcome before the carrier step, or have the carrier name the `archive/<slug>/` pattern on origin. Also state what the orch does when the teardown notice reports a failure.

3. MEDIUM · design · `decision-record.md` D4 steps 2 and 3 (their order).
   - Summary: the cancel, which cannot be undone, runs before the board write that moves the entry. If the new parent refuses the membership, the task is left a cancelled `draft` under its old parent and nothing has moved. A beam whose pawl run holds its entry is such a parent ("keeps the entry's members from joining"). So is a beam past `lym beam ready`, where a later member never starts.
   - Evidence: `internal/boardengine/runlock.go:49-53`; `contracts/orchrole/role-beams.md:61`.
   - Fix: check before the cancel that the new parent accepts a member, and name the refusals that stop the move.

4. MEDIUM · design · `decision-record.md` D4 step 3 (the discussed-entry branch) and D2's four sections.
   - Summary: the note is pasted under `## Notes for the plan writer` in the stored record. The discussion format allows no H2 sections besides the listed ones, so a note whose four sections are H2 headings, as a file of sections usually is, makes the seeded record malformed.
   - Evidence: `_bolt/_lym/specs/loom/loom-discussion-spec.md:13-25`; the seeded record is used as the run's own record: `internal/loomcli/discussionrecord.go:52-84` and `internal/shedrecipe/entries_block.go:22-28`.
   - Fix: give the note's sections a heading level, H3 or below or a bullet list, that is valid inside the Notes section.

5. LOW · consistency · `decision-record.md` D4 step 2 ("a cancel that reports a partial finish").
   - Summary: no cancel output field is called a partial finish. The orch cannot tell from the envelope when to run the cancel again.
   - Evidence: `internal/battencli/cancelcmd.go:154-174` (`end_only`, `way_forward`, `warnings`).
   - Fix: name the fields, for example `end_only` true or a non-empty `way_forward`.

6. NIT · consistency · `decision-record.md` D8's glossary line.
   - Summary: the line defines the move note as "what a cancelled run's writer learned", but under D3 the orch writes it.
   - Evidence: `decision-record.md` D3 and D8.
   - Fix: reword the line to "what a cancelled run learned".

**Tool calls:**
1. Grep for where a run reads a stored decision record (`.DecisionRecord(`, `Discussed`): the manifest did not show how a fresh run consumes a discussed entry's record.
2. Grep for `SeedDiscussedRecord` callers: needed to know what the seeded record skips.
3. Read `internal/loomcli/discussionrecord.go`: the seeding of the stored record into the run, which the manifest did not cover.
4. Read `internal/shedrecipe/entries_block.go:15-74`: confirmation that a seeded record passes over the Discussion block, which the manifest did not cover.
