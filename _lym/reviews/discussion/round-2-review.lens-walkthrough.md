```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision's mechanism, run end to end on one concrete case.

**Decisions walked**
- D1, a reader lookup for a function in package P that has a fresh worktree map. The worktree map wins for P, and other packages' maps answer from the index or the in-memory table. **Purpose met.**
- D2, a diff that moves a function verbatim to another file of its package and edits another function. The move is matched as unchanged and the edit is found. **Purpose met.**
- D3 at Darn-Write: the writer's verify passes, `reach.json` lists an unreached new function, and the gate returns the reach-turn failure. **Purpose broken**: the default cold fixer takes the turn, not the writer (F-W1).
- D3 at Code-Burler-Fix: the verify passes, the reach turn fires, the fix session adds tests and commits, and the next arrival passes. **Purpose met.** This entry has no fixer.
- D3 at Weaver's batch gate: the `batch` call passes, the turn reaches the batch strand, and that strand adds the tests. **Purpose met.**
- D4, a changed function reached by 600 tests across packages. The note ranks them, caps them at five plus a count, and gives the map commits. **Purpose met.**
- D5, Darn-Write's last gate call at HEAD H. The rubric fills at H, the report matches, and the review reads the note. **Purpose met.**
- D6, `lym gate reach internal/x/y.go#F`. It prints from the maps and writes nothing. **Purpose met.**
- D7, a red card gate whose failing test reaches none of the run's changes. The triage line, then the D8 command line, sit under the test, `FailedTests` parses as before, and the card fixer reads them on stderr. **Purpose met.**
- D8, the D7 line in a run worktree, with `--since` set to a last-green commit older than the merge base. **Purpose broken**: the walk runs to the parent's tip and names landings that are not in the failing tree (F-W2).
- D9, a diff with a verbatim cross-file move. The line diff marks the moved lines as added, so they get mutants. **Inconsistent with D2** (F-W3).
- D10 was not walked as a build: the added entry defers it.
- D11, `lym gate mutate --base <first parent>` in a replay worktree. It refreshes, plans and runs. **Purpose met.** That D11 prints "D10's report", which this build does not build, is left to the handler's added-entry check.
- D12, a hub with a tuned `batch` and no `reports`. Config load fills in `[reach]`. **Purpose met.**

**Findings**

- **F-W1 — BLOCKING, design.** Location: `_lym/discussion/decision-record.md:89-100` (D3, the reach turn and its rationale).
  - Summary: Darn-Write's verify entry is wrapped by a cold fixer that is on by default. The reach turn's failing result therefore spawns that fixer with the note, and the gate passes again on its re-run. The writer never hears the note, which defeats D3's stated purpose: "the writer … adds the missing tests itself".
  - Evidence: `contracts/recipes/darn-recipe.yaml:132-134` (the `verify` entry with `fixer: darn.gate_fixer`) and `internal/darnengine/template.yaml:5` (`gate_fixer: sonnet[medium]`, "clears a failing gate's mechanical findings before the writer hears of them").
  - Evidence: `internal/gatefixer/rowgate.go:58-67`. A failure spawns one fixer with the findings as its report, the closure runs again, and that verdict is returned. In D3, the re-run finds no new unreached function and passes.
  - Fix direction: D3 has to state how the reach turn gets past the row fixer, for example a result the wrap never hands to a fixer. Otherwise it should state that at Darn-Write the cold fixer gets the note.

- **F-W2 — MEDIUM, design.** Location: `_lym/discussion/decision-record.md:173-181` (D8, the walk range and the D7 command line).
  - Summary: the D7 command sits under a failing test in a run worktree, and D8 walks the parent's first-parent commits from `--since` to the branch tip. Landings after the run's merge base are not in the tree that failed, so the walk names suspects that cannot have caused the failure. D8 never bounds the walk at the merge base for that case.
  - Evidence: this is D8's own text, lines 173 and 176. The run's tree is the merge base plus the run's own changes; `parentMergeBase` is the base the redundancy step uses, at `internal/gatecli/redundancy.go:138`.

- **F-W3 — LOW, consistency.** Location: `_lym/discussion/decision-record.md:194` (D9, the changed lines).
  - Summary: D9 takes its changed lines from a line diff of the same base and worktree files as D2. A function moved verbatim to another file is therefore on added lines and gets mutants and survivors, while D2 calls it unchanged.
  - Evidence: D2 at line 60 against D9 at lines 189 and 194.

Tool calls: none.
