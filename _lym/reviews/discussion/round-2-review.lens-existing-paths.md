```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. For each state the decisions change, I traced the code that already writes or reads it and checked whether the decision's result survives.

**Paths traced, per decision**

- D1, record writers:
  - `add`, `store` and Bouncer `writeCarryOver` (`wiring.go:514`): survive, because the decision lock covers them.
  - The Discussion-Write and Discussion-Burler-Fix agents: survive, because replace is refused before approval.
  - Record-Write (`recordwrite.go:124`) writes only a missing file: survives.
  - Darn-Write's guard (`darnwrite.go:238-249`) only refuses and restores nothing: survives.
  - Shed status sweeps that commit `_lym`: survive, since D2 keys the cut on the bytes' hash.
- D1, record readers:
  - Plan-Write and drift seats (`driftengine/seats.go:104`): survive, since folded text is read as main decisions.
  - Webster Master's `parent_decisions` (`websterengine/runlevel.go:734`) is read at every fresh Master spawn: **breaks** for a fold made before the Webster rows (F1).
  - Bouncer `addedSinceSnapshot` (`bouncerdecision.go:316`): survives; the Open risks name it.
  - Report `readDecisions` (`reportengine/rounds.go:239-259`): **breaks**; it misreads a folded record (F4).
  - Raddle-File (`loomshed/raddlefile.go`) re-checks the proposal against the record: survives for the checks, but **breaks** on the hash-skipped steps (F2).
- D2, history readers:
  - `ValidateDiscussion` (`gates.go:157-162`), `checkDarnRecord` (`decision.go:470`), the darn guard (`darnwiring.go:165`) and validate-discussion all go through `recordHistory`: survive, because D2 says every caller reads the bounded history.
  - Other `FindTrailerCommit` key searches: survive, since the key is new.
- D3, the board's stored record:
  - Record-Write seeding from the board: survives (Open risk names the re-seed case).
  - `promoteTo`'s `recordStored`: survives.
  - `FileRaddle` part and project record writes: **breaks** in the window between the board save and the store's done marker (F3).
  - `EndRaddleEntry` followed by the refusal for a done entry: survives.
  - Batten cancel clears status and keeps the record file: survives.
- D4, `decision-<slug>.md` and `board.json` writers:
  - `StoreDecisionRecord` and `FileRaddle`: survive, since D4 refuses a held entry and a slug with a worktree.
  - `followDecisionRecords` rename and removal: survives.
  - Bolt replay (`replay.go` claims `decision-*.md`): survives, because file and entry travel in one commit.
  - Record-Write's never-overwrite rule: survives via the worktree refusal.
- D5, `record_body_sha256`:
  - Body writers: upsert and merge leave the hash, so the warning fires as intended.
  - `arrasProjectOf` restamps through Raddle-File: survives.
  - Sketch conversion (`endsAsSketch`, no body change): survives.
  - Replay merges the entry by field: survives.
  - An older binary keeps the key in `Unknown`: survives.

**F1 — MEDIUM — design — D1, Webster-row refusal**
- A replace that folds `Added after Discussion` entries after plan approval but before the run reaches Webster removes them from every Master that later starts.
- Master reads no record. `websterengine/runlevel.go:734` builds `parent_decisions` from `discussionparser.AddedDecisionTexts` at each fresh spawn, and `websterengine/doc.go:338-341` says Master learns a post-plan decision only that way.
- D1 refuses only while the current row is a Webster row. A call recorded with `add` (no `--amends-plan`) after Plan-Write, then folded at a Drift row or Plan-Review row, reaches no Master.
- The same holds for a later goto back to Webster.
- The record is silent on runs past plan approval that have yet to reach or re-enter Webster.

**F2 — MEDIUM — design — D1/D3, replace on a non-perch raddle run standing at Raddle-File**
- A replace made after Raddle-File staged its store files and part records, then halted, never reaches the filed parts.
- `loomshed/raddlefile.go:135-137`: "A step whose outputs carry the hash of the proposal now approved is skipped". The staged `SketchRecord` (`:481`) and the part records are keyed to the proposal's hash, not the record's.
- A halted Raddle-File (step 3 Stuck, `:133`) followed by a replace that keeps the ids passes the re-check (`:267`). The resume then skips staging and the part records and files the parts with the pre-replace design.
- Meanwhile D3 copies the new record onto the Sketch.
- `perchFilingRefusal` (`decision.go:942-958`) blocks this only for perch mode. D3 extends the refusal only once the filing is done.

**F3 — MEDIUM — design — D3, refusal keyed on `raddleRunFiling`**
- In a family filing, `FileRaddle` saves `board.json` and the project's record file first, and writes the store's done marker afterwards (`boardengine/raddlefile.go:147-150`, `:337-361`).
- `raddleRunFiling` reports done only when the store holds the filing (`loomcli/raddlewiring.go:82-97`).
- If the store write fails or the process crashes in between, the run halts at Raddle-File with done = false. A D3 `add`, `replace` or `store` is then admitted and copies the Sketch's record over `decision-<slug>.md`, which is now the project's record (`convertRunEntry`, `:214-226`).
- `StoreDecisionRecord` does not check the run lock, and the converted entry has no status anyway.
- The Raddle-File retry takes the held-filing path and writes only the store (`raddlefile.go:117-119`), so the overwrite stays.
- Today the raddle recipe has no approval row, so `copyRecordToBoard` never copies. D3 opens this path.

**F4 — LOW — design — D1/D2, a fold vs the run report's decision counts**
- `reportengine/rounds.go:237-258` counts post-Discussion decisions per `--by` author from the archived record's `Added after Discussion` entries.
- After a replace that folds them, the report shows zero or fewer decisions for that run, though D2's bound keeps the replace's `--by` only in the trailer.
- The record does not say whether the report should read the `Lym-Record-Replaced` trailers or accept the undercount.

**Tool calls**
1. `loomshed/darnwrite.go` restore paths: the manifest lacked Darn-Write's record restore code.
2. grep for raddleparser check callers and Master `parent_decisions`: to find where the record is re-read at Raddle-File and Master spawn.
3. grep of `loomshed/raddlefile.go` for record and hash lines: Raddle-File's step-skip logic was not in context.
4. `loomshed/raddlefile.go` doc and verify span: to confirm that hash-keyed steps are skipped and that the checks re-run.
5. `reportengine/rounds.go` `readDecisions` and the websterengine `parentDecisions` source: one more reader of added entries, plus Master's source line.
