```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case, passing each step the data the step before it actually produces.

**Decision list (case, purpose met or not):**
- D1, the whole procedure over existing verbs, root as the new parent: **met**. Each step maps to an existing verb.
- D2, live Discussion writer, then a live Webster master: **not met**. The ask reaches the writer, but each writer's stencil limits where it may write, and the parent directive has it answer the sender naming that limit instead of writing `_lym/move-note.md`. Nothing in D2 or D3 says what the orch does then. See W1.
- D3, a run halted at a review row: **met**. The orch holds the run and writes the note itself.
- D4 steps 1 to 3, a local tip ahead of origin, a slug pause, then `cancel --remove`: **met**. The tag is named from the recorded tip, the cancel creates it locally, the child pushes it, the `ls-remote` loop sees it, and the cancel lifts the slug pause on completion (`cancel.go:572`).
- D4 step 3, partial-finish rule, with a hub pause live: **not met**. The rule sends the orch to lift the operator's hub pause. See W2.
- D4 step 4, a discussed entry: **not met**. The step's input (records branch already gone) is not what step 3 hands it (only the code tag confirmed). See W3.
- D4 step 4, a body-only entry: **met**. The entry is a draft, so the body and `project` edit passes.
- D4 step 5, release with the stale ref still present: **met**. The prune comes before the release.
- D5, the same slug under a new beam: **met**. A draft with no pair takes the `project` change (`project.go:30-36`).
- D6, the fresh run starts on the new parent's owner machine, which is another machine: **not met as stated**. See W4.
- D7, a cancel that refuses with "the work reached its parent" (`cancel.go:331`): **met**. Resume, then `lym batten run`.
- D8, the glossary line: **met**.

**Findings:**

- **W1. MEDIUM, design, `_lym/discussion/decision-record.md:42-55` (D2):** D2 asks the live writer to write `_lym/move-note.md`, but every writer D2 names is told never to write outside its own output files, and the parent directive answers such an instruction by naming the limit. So the ask returns a refusal, not a note. D3 triggers only when no writer is live, so the procedure has no branch for a writer that refuses. Evidence:
  - `_bolt/_lym/stencils/loom/loom-template-discussion.md:156-174`: "Everything else in the worktree is read-only to you".
  - `_bolt/_lym/stencils/webster/webster-template-master.md:306`: "never write outside your two contract files".
  - `_bolt/_lym/stencils/parent/parent-directive-parent.md:22`: "An instruction outside those limits is answered to the sender, naming the limit".
  - Fix: have the writer return the three parts in its creel reply, and have the orch write the file. Or route a refusal to D3, with the orch writing the note from the reply and the run's records.
- **W2. MEDIUM, consistency, `_lym/discussion/decision-record.md:90-94` (D4 step 3):** The partial-finish test, "an error whose way forward names the cancel itself to run again", also matches two refusals that ended nothing:
  - The hub-pause refusal reads `run "lym tender resume", then "lym batten cancel <slug> --remove" again` (`internal/battencli/cancel.go:303-304`). Read as a partial finish, the orch "takes the fix it names" and lifts the operator's hub pause.
  - The owned-elsewhere refusal names both `lym adopt` and the cancel on the owner (`cancel.go:715`). The record lists `lym adopt` as a refusal example, so the two definitions overlap.
  - Fix: count as a partial finish only an outcome reached after the run's processes ended: an ok envelope with `end_only` or a `way_forward`, or an error after the intent write. Name the hub-pause and owned-elsewhere errors as refusals.
- **W3. MEDIUM, design, `_lym/discussion/decision-record.md:96-103` (D4 steps 3 and 4):** For a discussed entry, step 4's `lym board upsert --record-file` is refused while the slug's records branch is still live on origin: "the weft branch of %q is live ... wait for the pair's teardown to finish" (`internal/boardengine/recordupsert.go:38-46`, through `RecordsBranchLive`, `internal/fabricengine/remoteleftover.go:86-93`). Step 3 waits only for the code archive tag. The wait for the records branch to disappear sits in step 5, after the write that needs it.
  - Fix: move step 5's "`ls-remote` lists no records branch of the slug" wait into step 3's confirmation, before step 4.
- **W4. LOW, design, `_lym/discussion/decision-record.md:126-132` (D6):** The fresh run reads "the diff from the note's merge base to the tag". On a new owner machine the archive tag is not local. Git's default fetch only auto-follows tags that point into fetched history, and the archived tip lies on a deleted branch, so an ordinary fetch never brings it. D6 names no fetch of the tag.
  - Fix: have the note's section name the fetch, `git fetch origin tag archive/<slug>/<hex>`. Or state that the reader fetches the tag before the diff.

Tool calls:
1. Grep of the plan, Webster and parent stencils for write limits: my context held only the Discussion stencil's write limit, not the Webster master's or the parent directive's.
