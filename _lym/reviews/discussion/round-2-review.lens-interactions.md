```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. Check each pair of the record's decisions that touch the same state and report where one breaks another.

**Pairs checked**
- D1 × D6 (a restart after the filing meets the `undiscussed` gate on the converted entry, which stays discussed): held.
- D1 × D7 (`decision add` finds the Motif block's Discussion approval row): held.
- D2 × D4 (on a return, the proposal is moved aside as in auto mode and the perch is re-armed): held for a Raddle-File return.
- D2 × D4 (the fixer's `raddle` gate pass stores the cold proposal only once, so the cold proposal stays the first one that passed): held.
- D2 × D5 (the fixer's gate applies the perch-mode outcome checks): held.
- D2 auto-accept × D4 Bound (auto-accept can file a round that still had open findings; Raddle-File's checks and draft status still apply): held.
- D2 re-arm × D7's way forward before the filing (`lym loom goto --to Raddle-Write`): broken, F2.
- D4 × D7 (the refusal keys on the `perch` mode D4 records at Raddle-Write's first call, before Raddle-File): held.
- D5 one-task × D10 (dependents re-pointed in the filing write, entry ended done by `EndRaddleEntry` afterwards): held.
- D5 family × D6 (project slug equals the entry; `slug-taken` exempt; a part's dependency on the run entry dropped, as `fileRaddleEntry` does): held.
- D6 run-lock admission × D10 (D10 changes only other entries; a run-held dependent is refused and the run halts): held.
- D6 failure path × D10 (a refused or failed write saves nothing, so dependents and the record stay as they were): held.
- D6 retry × D10 (`raddleFilingHeld` matches the converted entry by `RaddleProjectBodyPrefix` and the discussed flag; dependents are already re-pointed): held.
- D6 status skip × D6 retry × D8: broken, F1.
- D6 × D8 (a converted project with no status is released with a null `set-status`): held on the no-crash path.
- D7 × D3 / Open risks (a decision added between the perch's approval and Raddle-File is not reviewed): consistent with the stated risk.
- D8 × D6 criterion (the filing write and null statuses written by upsert, merge or claim clear release no member): held.
- D8 × D10 (released members start in `depends_on` order; dependents wait on the last parts): held.
- Constraint `PATTERN-shed-recipe-registry` × D4 and D7 (the refusal is decided by the rows, through the mode): held.

**Findings**

F1 — MEDIUM, design. Location: `decision-record.md` D6, lines 134–138 (the post-filing status skip and the retry).
Summary: a crash between the filing write and the filing-mark write leaves the converted project with a run status that nothing clears. D8 then refuses the release, and `RunHolds` holds the project's members.
- The record says Raddle-File writes the mark "when its board write lands with a project". That makes the mark a separate write after `FileRaddle` saves.
- On resume, the loom status hook's first transition is into Raddle-File. Its `lastBoardStatus` starts empty in the new process, so it calls `SetRunStatus("running · Raddle-File")` (`internal/loomcli/wiring.go:309-325`). `SetRunStatus` skips only an absent or done entry (`internal/boardengine/board.go:221-230`), and the run lock does not judge an entry with no status (`internal/boardengine/runlock.go:158-160`). The run status therefore lands on the converted project.
- The retry then "writes only the store" (D6). It never says it writes the mark or clears that status. The family outcome has no `EndRaddleEntry`, and `finishesAsSketch` is false for a recipe with a RaddleFile row, so the Done transition writes nothing either (`internal/loomcli/wiring.go:241`, `312-318`).
- Result: the project keeps "running · Raddle-File". D8 refuses its release because it has a run status. `RunHolds` is true for a project with a run status that is not a beam (`runlock.go:55-61`), which keeps its members from running inside it.
- The record's guarantee, and the criterion "no status write of the arras run after the filing changes the converted project", break on this path.
- Fix: D6 states one of these:
  - the mark is written before the board write and removed when that write does not land;
  - the retry that finds the converted entry writes the mark and clears any run status on the entry before the row ends.

  Add a criterion for a resume after a crash between the two writes.

F2 — MEDIUM, design. Location: `decision-record.md` D2, lines 62–63, against D7, line 152.
Summary: D7's way forward before the filing is a goto into Raddle-Write, but D2 re-arms the split perch only after a Raddle-File return. The re-split made after a recorded decision can reach Raddle-File without a perch review.
- `freshRaddleEntry` treats a goto routed to Raddle-Write the same as a Raddle-File return (`internal/loomcli/raddlewiring.go:57-66`). D2 states the clearing of the settle record, and the fresh cycle with full budget, only for "a return from Raddle-File". The D2 criterion likewise covers only "after a Raddle-File return".
- With the goto, the perch's settle record from the earlier approval can stand. The new proposal, which carries the decision D7 just recorded, then settles as already approved and files unreviewed. That defeats D7's stated purpose: "re-splits with the decision recorded first" so the perch reads it.
- D4, line 88 ("A return from Raddle-File moves the last proposal aside") has the same return-only wording, though the code clears on a goto too.
- Fix: D2's re-arm, and D4's set-aside, apply to every fresh entry into Raddle-Write, a goto included. Extend the D2 criterion to a goto.

Tool calls: none; my inherited context held every definition these findings rest on.
