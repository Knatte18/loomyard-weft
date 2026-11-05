```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge each of the reviewer's proposals the fixer carried out (F1, F2, F3) by building concrete cases where the proposed change is wrong.

Proposals challenged: F1 (records trailer), F2 (snapshot move, untracked copies, lost-snapshot rule), F3 (baseline digest recomputed after the judge).

1. **MEDIUM.** D9, the new sentence "When the snapshot is gone … Drift-Verify takes a new snapshot before it resumes or respawns the judge and drops the Stuck for the earlier session". Summary: under this rule, a judge can defeat its own tree guard.
   - Case: the judge edits a tracked source file in the warp checkout. It then deletes the snapshot directory by absolute path.
   - D9's new Bound says "files outside both checkouts … are outside the guard", and the snapshot now sits under the hub's board-level `.lym`, outside both checkouts.
   - The session returns, Drift-Verify finds the snapshot gone and takes a new one that already contains the edit. The Stuck is dropped, and the judge's code edit stays in the worktree with no stop.
   - The same happens after any `.lym` clean while a judge mark is open.
   - D9 states no bound for this, and the earlier Stuck it drops is the only evidence. Before this fix the record gave no lost-snapshot rule, so this pass-through is new.
   - A safer rule: a snapshot missing while the judge mark is still open is a Stuck for the parent or operator, never a silent drop.

2. **MEDIUM.** D6, the new sentence "A run with no commit carrying `Drift-Records`, seeded before this lands, reads the files as they are until its first records write makes one", read together with "a row that writes the records starts from those copies". Summary: in-flight runs keep the bypass, and the rewritten Bound does not say so.
   - Case: a run is at its writer row when this lands, so it has no `Drift-Records` commit yet. The writer hand-adds an amendment or sets `Deferred` in `scope.json`.
   - The first Drift-Verify reads the files as they are, with no copies to start from, and commits the forged values into the first `Drift-Records` commit. From then on they are guarded as genuine.
   - D6's Bound still says "forging a seal or a drift record needs a hand-made weft commit". Its in-flight clause covers only a forged seal, not forged records.
   - D7 states its matching in-flight gap for the digest ("checked without this guard"). D6 states none for the records, and Open risks does not list it either.

3. **LOW.** D6, "`Drift-Records: <sha256 over scope.json and judged.json>`". Summary: the hash is undefined when `judged.json` is absent, so deleting it may go unnoticed.
   - Case: a run in `command` mode, or one that has not yet been judged, has no `judged.json`. A writer deletes a `judged.json` that holds a stored `does not match` ruling.
   - If the hash treats an absent file as empty, deleting a file that existed changes the hash and is caught. Treating it as empty is one possible reading, and the record does not say which applies.
   - If a reader takes an absent `judged.json` to mean "no records", the stored failing rulings are lost. A resumed row then calls a new judge instead of stopping, which defeats D2's and D10's rule that a stored failing ruling stops a resumed row without a new judge.
   - The record should state that the hash covers each file's presence as well as its content.

4. **LOW.** D6, "A records write interrupted before its commit shows as a mismatch, and `--restore` drops it". Summary: a crash inside lym now needs the parent to act.
   - Case: Drift-Verify is killed between writing a passing check result and committing it. On resume the row is Stuck naming the parent's `lym drift amend --restore`, so `lym loom resume` alone no longer recovers.
   - The way forward exists, so this is a cost, not a dead end. Still, nothing in D4 or D12 marks this as a new Stuck that only the parent can clear.

F3's proposal: I found no breaking case. A judge that edits the baseline leaves `defer` as the only way forward, but D7 already accepts that for a writer, so it is not new.

Tool calls:
- grep under `internal` for code that writes `scope.json`, to check whether anything outside the drift rows writes the records. My context did not have the code's write sites. Only `internal/driftengine/records.go` (`UpdateScope`) writes it.
