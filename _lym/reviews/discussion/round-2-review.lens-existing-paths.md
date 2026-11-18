```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens. I traced the existing code paths that meet the state each decision introduces or changes.

**Paths traced, per decision**
- D3, Darn-Write verify gate (`loomshed.NewVerifyGate` under `gatefixer.WrapRowGate`, `fixer: darn.gate_fixer`): **breaks**. The reach turn's failing result goes to the cold fixer, not to the writer.
- D3, Code-Burler-Fix verify gate (no fixer element): survives. Its failing result re-prompts the fix session.
- D3, weaver batch gate (`batchGateClosure`, `batchGateAttempts`): survives in the main, with one caveat. The same closure also serves each `card <id>` entry, and the record only names "the weaver's batch gate".
- D3, Verify skip path (`StatusSkipped` on an already-verified tree): survives. A later arrival on an unchanged tree reads the existing `reach.json` and passes, because the recorded list holds the functions.
- D3, darn's failed-tests-alone rerun (the `gateslot.FailedTestsCall` call carries no `--gate`, so it writes no report): survives. The full call that follows writes `reach.json`.
- D3, anchor maps against `lym gate test --map` and `lym gate redundant` in the same worktree (`coverageMapsDir`): survives. Both read and write the same anchor directory with the same format, and the redundancy step reads only the hub maps.
- D3, the `--map` write checks (HEAD moved, dirty owned or linked file): survives. D3 pre-checks both, and `verifytree.Verify` refuses a dirty tree.
- D3, Verify's prune of verify logs: survives. `pruneLogs` removes only `verify-<n>.log` files, so a listed-functions record in the verify directory is left alone.
- D3, gate closure state across a driver restart: survives. The attempt count lives in memory, as `PassOnCap`'s count does, and the listed functions are persisted.
- D7, `gateslot.FailedTests`: survives. Four-space lines are skipped as failure output.
- D7, the weaver verify fixer (`runplan.ParseVerifyRun` parses `go test` output, not the failure report): covered by D7's "a fixer whose report file does not copy the failure report" clause.
- D7, the Darn-Write cold fixer report (the findings of `verifyFailureFindings`, quoted from `go test` output): covered by the same clause.
- D12, the hub's tuned `darn-write`, `code-burler-fix` and `batch` entries, which hold no `reports`: survives. Config load fills the template's `reports` into a tuned entry that lacks them.
- D5, the rubric marker filled at each fill as `redundancyReportNote` is: survives.

**Findings**

1. BLOCKING · design · `_lym/discussion/decision-record.md:89-98` (D3, the reach turn)
   - **Summary:** in Darn-Write the reach turn's failing result reaches a cold fixer, not the writer. That defeats D3's stated purpose, "the writer get the note from its own gate before review, so it adds the missing tests itself", and the record says nothing about it.
   - **How it breaks:**
     - Darn-Write's verify entry carries `fixer: darn.gate_fixer` (`contracts/recipes/darn-recipe.yaml:132-134`), and the default is `sonnet[medium]` (`internal/darnengine/template.yaml:5`).
     - `WrapRowGate`'s spawning closure hands any non-passing, non-pending, non-terminal failure to a cold fixer. It writes the findings as the fixer's report, then runs the inner gate again (`internal/gatefixer/rowgate.go`, `spawning` and `spawnAndJudgeAgain`).
     - That second inner run finds no unreached function that no earlier turn listed, so it passes or fails as the verify does (D3, line 95). The writer never sees the note.
     - Each turn also spends a fixer spawn under the wrap's attempts+1 cap.
   - **Fix:** in D3, state how the reach turn meets the row gate's fixer wrap. For example, mark the reach-turn result so the wrap skips the fixer, or have the reach turn bypass the fixer entry. Add an acceptance criterion that Darn-Write's writer, not its cold fixer, receives the note.

2. LOW · design · `_lym/discussion/decision-record.md:89-90` (D3)
   - **Summary:** "the weaver's batch gate" is built by `batchGateClosure`, which also builds every `card <id>` entry. Those entries run `--gate card`, which writes no `reach.json`. After a pass they would read a `reach.json` an earlier `batch` call wrote, and the record does not say whether they take the turn.
   - **Evidence:** `internal/weaverengine/strand.go:138-176`.
   - **Note:** this is below the incremental round's BLOCKING-only bar for new findings. It is listed for the handler's judgment.

**Tool calls**
- `internal/gatefixer/rowgate.go` lines 80-200: the context held only the head of the wrap, not its spawn-and-rerun path.
- The hub's `_bolt/_lym/config/gate.yaml` gate keys: to check whether the tuned entries hold `reports`, for the template fill in D12.
- `runplan.ParseVerifyRun`: to see whether weaver's verify fixer report is built from the gate's failure report, for D7.
