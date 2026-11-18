```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I tried to break every bound, guarantee and claim about existing code in the decision record. One claim breaks badly (D3's reach turn on Darn-Write), and a few minor bounds break or are worded wrong.

**Claims attacked**
- D3 rationale, "the writer get[s] the note from its own gate … so it adds the missing tests itself", for Darn-Write: **broken**. Darn-Write's verify gate hands every failure to a cold fixer first.
- D3, the reach turn spends one attempt, and the gate passes without the turn when one more failure would reach the budget: **broken** under the cold-fixer wrap, because the inner closure runs twice per arrival.
- D3, `verifytree.Verify` refuses a dirty tree, so those gates' packages are always refreshable: held for dirt. HEAD moving mid-call is a separate gap (finding 4).
- D3, "a function listed once never brings another turn", with the next arrival passing on a Skipped verify: held.
- D3, the reach step never changes the exit of the `lym gate test` call: held.
- D3, the refresh runs "with the `--map` measurement, inside a gate slot": **broken**. The `--map` run holds the larger of tier0's and tier1's slot asks, not one slot.
- D3, the linked packages come from the package's map: **broken** when the diff adds an import.
- D3 note, the reasons a package is left unrefreshed (uncommitted files, past the budget, in the prime): **incomplete**. A moved HEAD also leaves packages unmeasured.
- D1, the index is written only by the prime's map run, so the shared directory stays prime-written: held.
- D1 bound, a same-size rewrite within one modification-time tick keeps its stamp: held; the bound is stated.
- D1 bound, "a map run that ended before its index write, as a moved HEAD ends it": **example wrong**. A moved HEAD ends the package walk, not the run.
- D1, the worktree map wins only when its file hashes match: held.
- D5, matching on test files means a report written before a new test stops matching: held. The bound for unnamed packages is stated.
- D7, `gateslot.FailedTests` parses the report unchanged with four-space triage lines: held; four-space lines are skipped.
- D7, the card fixer's report is the call's stderr (`fixAndRerun`): held; the function is in `internal/gatecli/detachfixer.go`.
- D8, the 200-commit cap, `--since` required, and lym records no last green commit: held.
- D9, mutants on every changed line of covered blocks, uncovered blocks reported: **broken** for changed lines outside any block.
- D9, a timeout or memory-cap run is unjudged, as in `lym gate redundant`: held.
- D12, an existing hub gets the template's `reports` with no reconcile: held per `internal/gateslot/doc.go:113-114`. A hub entry with an explicit `reports` list keeps its own list, but I found no hub file holding one.

**Findings**

1. **BLOCKING, design.** `_lym/discussion/decision-record.md:89-103` (D3 reach turn and rationale).
   - **Summary:** Darn-Write's verify gate is wrapped with a cold fixer, so the reach turn's failing result never reaches the writer. The fixer gets the note, the wrap re-runs the closure, and the re-run passes because the function is now listed. The operator's purpose (the writer adds the missing tests itself) fails, and every reach turn spawns a cold fixer.
   - **Counterexample:** Darn-Write commits a new unreached function. The verify passes, and the gate returns the reach turn's failing result. `WrapRowGate` spawns the `darn.gate_fixer` fixer (default `sonnet[medium]`) with the note as its report and waits for it. It then runs the closure again over the fixer's tree. If the fixer committed nothing, Verify is Skipped, the function is already recorded as listed, and the gate passes. The writer never hears of it.
   - The same wrap breaks D3's attempt count. "Counts its own failing results since its last pass" sees the inner closure run twice per shuttle attempt, so it no longer matches the entry's re-prompt budget.
   - **Evidence:** `contracts/recipes/darn-recipe.yaml:127-134` (`fixer: darn.gate_fixer` on the verify entry); `internal/darnengine/template.yaml:5` (`gate_fixer: sonnet[medium]`, "clears a failing gate's mechanical findings before the writer hears of them"); `internal/gatefixer/rowgate.go:59-72` (`WrapRowGate`: a failure spawns one fixer, then the closure runs again and its verdict is returned).
   - **Fix:** D3 says how the reach turn passes the row-gate fixer. One option is a result the wrap does not hand to a fixer, as `SkipForked` already does for findings meant for the writer alone. D3 then states how the attempt count is kept under the wrap, and the D3 acceptance criterion covers a Darn-Write gate with `gate_fixer` set.
2. **LOW, consistency.** `_lym/discussion/decision-record.md:75` (D3 first bullet) and Constraints line 252.
   - **Summary:** "With the `--map` measurement, inside a gate slot" contradicts the `--map` run's own slot rule. That run holds the larger of tier0's and tier1's slot asks, `slots: all` included, and runs each tier's tests up to its parallel side by side.
   - **Counterexample:** a hub whose tier1 asks for `slots: all`. A one-slot refresh then runs those tests beside other holders.
   - **Evidence:** `internal/gateslot/doc.go:183-185`; `pattern/PATTERN-test-speed.md:302`.
   - **Fix:** D3 says whether the refresh takes the `--map` slot ask or one slot with tests run one at a time, and the D3 cost bound matches that choice.
3. **LOW, design.** `_lym/discussion/decision-record.md:80` (D3, how linked packages are found).
   - **Summary:** For a mapped package, the linked set is read from its existing map, which predates the diff. A diff that adds an import of a package with uncommitted files passes D3's check. The `--map` write then refuses it: budget spent, no map written, and the note gives no reason.
   - **Evidence:** `internal/gatecli/testmap.go:584-605` (`checkClean` checks the packages the current build links).
   - **Fix:** take the linked set from the Go toolchain's dependency list of the current tree for every changed package, or name this case in the note.
4. **LOW, consistency.** `_lym/discussion/decision-record.md:82-84` (D3, the reasons a package is left unrefreshed).
   - **Summary:** A HEAD that moves during the refresh, such as a commit made while a writer's hand-run gate is going, ends the `--map` walk. The remaining packages are left unmeasured, and D3's note has no reason for that.
   - **Evidence:** `internal/gatecli/testmap.go:532-563` and `:578-581`.
   - **Fix:** add "HEAD moved" to the reasons the note names, with the way forward to run the gate again.
5. **LOW, design.** `_lym/discussion/decision-record.md:189-195` (D9).
   - **Summary:** Mutants go only on changed lines inside covered blocks, and only changed blocks are reported uncovered. A changed line outside every block is neither mutated nor reported, so it looks checked: a package-level `const`, a `var` initializer, or an embedded file's line.
   - **Evidence:** `internal/testcover/doc.go:47-55` (the planner reaches package-level constant and variable values and embed lines only through the blocks that use them).
   - **Fix:** D9 either places a mutant for a changed declaration at the covered blocks that name it, as the existing planner does for constants, or reports such a line as not judged.
6. **NIT, consistency.** `_lym/discussion/decision-record.md:54` (D1 bound).
   - **Summary:** "As a moved HEAD ends it" is the wrong example. A moved HEAD ends the package walk (`mapPackages` breaks and lists the rest unmeasured), not the run, so the index write after the maps can still happen.
   - **Evidence:** `internal/gatecli/testmap.go:526-563`.
   - **Fix:** name a cause that ends the run before its index write, such as a cancelled call.

Tool calls:
- One grep: `fixAndRerun` in `internal/gatecli`, to check D7's claim (the card fixer code in my context did not hold it); `reports` in the hub's `gate.yaml`, to test D12 (no hub file was in my context, and the grep found nothing); and the verify rows of the darn and tapestry recipes, to see which rows reach `loomshed.NewVerifyGate` (only the darn rows were in my context).
