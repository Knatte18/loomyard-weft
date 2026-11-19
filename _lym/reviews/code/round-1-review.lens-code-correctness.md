```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran the diff's changed functions with concrete inputs and found one design defect, one low-severity gap and one low-severity memory risk.

**Per function or branch: case run, verdict**
- `verifytree.WithReachTurn`, two inner failures then a pass at attempts 3: the count reaches 2 and the pass skips the turn. Correct.
- `WithReachTurn`, attempts 3 with one prior failure: the turn runs and counts 2. Correct.
- `WithReachTurn`, rerun after a fixer: the failure counts once per arrival. Correct, and a pending error is returned. The listed-file defect below is a separate case.
- `WithReachTurn.findings`, listed set shrinks: see F1.
- `gatefixer` `rowWrap.spawning` and `spawnAndJudgeAgain`: a `WriterOnly` result spawns no fixer, and the re-run goes through `w.rerun.Run`. Correct. A nil mark is safe.
- `rowWrap.waitThenRun` on a live record: see F2.
- `RerunMark.Run` and `On`: nil receiver, nested calls and the atomic counter. Correct.
- `testreach.ChangedFuncs`: new function, body change, moved file, build-constrained variant and two `init`s. Correct. A comment-only change is not changed, because the hash excludes the doc.
- `ChangedLines`: empty base, trailing newline, prefix and suffix trimming. Correct. See F3 for large middles.
- `testreach.Open` and `worktreeMapWins`: worktree map fresh, stale and later, and no worktree directory. Correct, and `TestsReaching` filters to the winning map's entries.
- `readSharedTable`: a current index serves it and an absent or stale one builds in memory. Correct. `WriteIndex` stamps before reading. Correct.
- `FuncsReachedBy` and `StaleLinkers`: a subset, an absent test, and a linker with older hashes. Correct.
- `fabric.FirstParentCommits`: range exactly at the limit returns met; over the limit returns limit+1 and false; `since` equal to the tip returns empty and met; history ending without `since` returns false. Correct.
- `runReachTest`: root package `..TestX` splits at the last dot. Correct. A root commit (empty parent) is skipped. Correct.
- `reachRequest.formRefusal`: both, neither, `--since` without `--test`, and `--test` without `--since`. Correct.
- `ReachReportMatches`, `ReachPendingOf`: red, prime, note, other key and stale HEAD give false. Correct. `ModuleDir` ".": `path.Join(".", "go.mod")` is "go.mod". Correct. Reads and matches that fail Warn and answer false. Correct.
- `reachStep.refreshMaps`: the prime, a dirty own or linked package, a spent budget, and a missing map via `LinkedPackages`. Correct. See F4.
- `triageFailuresKept` and `reportLines(triageKept)`: a triage that does not fit whole gives none; kept entries are a prefix. Correct. `TriageLines` and `FailedTests` are unaffected.
- `triageLookup.lines`: unmapped, no change set, none reached with the command, more than five, and a subtest cut at the first slash. Correct.
- `PlanLineMutants`: one mutant per line by `operatorRank`, strict `<` keeps the first on ties, and IDs are renumbered. Correct. A logger call, a string and an embedded line get none. `callee` is set right after `add`, which always appends, so `sites[len-1]` is safe.
- `blockHoldsLine` bounds: lo ≤ hi always holds because the line lies in the block's range, so no slice panic. Correct.
- `diffMutation.run` and `selectWithinCap`: round-robin over function groups, a cap of 1, and `NotRun = total - taken`. Correct.
- `diffMutation.planPackage` and `judgeMutants`: `holdSlot` runs first and sets `tmp` and `sideBySide`, so a copied check never has zero workers. Correct.
- `diffMutation.scan` and `record`: a kill re-run alone, a timeout or memory cap leaving the mutant unjudged, and budget stops not counted as run. Correct.
- `runMutate`: no parent and no base, a bad base, the prime with a base, and an empty change set. Correct.
- `gateslot` config decode: each new key below 1 is refused and `reach` is accepted. Correct.

**Findings**

- **F1 — MEDIUM, design.** `internal/verifytree/reach.go:160` (`findings`): after a turn, `WriteJSON` replaces the listed file with only the current `pending.Functions`.
  - Case: turn 1 lists {A}. Later A is reached and pending is {B}, so the turn lists {B} and drops A. If A is unreached again with the same hash, it brings another turn.
  - D3 says "a function listed once never brings another turn".
  - Fix: write the union of the old `listed.Functions` and `pending.Functions`, updating the hash for an id whose hash changed. Add a test row for the shrink-then-regrow case.
- **F2 — LOW, design.** `internal/gatefixer/rowgate.go`: the live-record path `return w.waitThenRun(store, live, inner, true)` runs the inner gate without the re-run mark.
  - Case: after a driver restart finds a live fixer, the judging run counts a failure in `WithReachTurn`.
  - D3 says the wrap marks that second run.
  - Fix: wrap it as `func() (...) { return w.rerun.Run(inner) }` on that path too.
- **F3 — LOW, design.** `internal/testreach/changedlines.go:386`: the LCS table is O(n·m) in the trimmed middles.
  - Case: a rewritten 10,000-line file allocates about 100M ints in `lym gate mutate`.
  - Fix: cap the middle size, with a fallback that marks every head-middle line as changed.
- **F4 — LOW, design.** `internal/gatecli/reach.go`, `refreshMaps`: when `run.fabric.HeadSHA()` fails it returns after recording only an Errors line, with no `unrefreshed` line per package.
  - The note then answers from old maps with no per-package reason, which D3 requires for each unrefreshed package.
  - Fix: record `unrefreshed(pkg, "HEAD could not be read; way forward: run the gate again")` for every package before returning.

**Tool calls**
1. `grep` for `goFiles` and `workingTree` in `internal/gatecli`: found nothing because the receiver form differs. Needed to check missing-directory handling; the context lacked it.
2. `grep` on `internal/testcover/mutant.go` and `redundantmutants.go`: confirmed `add` always appends, and found where `sideBySide` and `tmp` are set. The context lacked both.
3. `grep` for `goFiles` and `holdSlot` in `internal/gatecli`: located `goFiles` at `redundancytests.go:49` and confirmed `holdSlot` sets `tmp` at line 550. The context lacked the location.
4. `Read` of `redundancytests.go` lines 40-110: showed `workingTree.goFiles` returns nothing for a missing directory. The context lacked it. `commitTree.goFiles` through `CodeDirAt` on a missing base directory was not checked.
