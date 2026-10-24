```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness.** For each function or branch, the case I ran and whether it was correct:

- `greenRecord.with`: the same-key whole entry replaced a whole entry, a package entry joined a package entry, and a package entry arriving over a whole one returned the record unchanged. Pruning kept the current tree plus each fingerprint's newest whole commits. Correct, with one wrinkle (finding 3).
- `greenRecord.covered` and `argsMatch`: an empty-args entry covered an empty request and a `-run ^(…)$` request. A guard-args entry did not cover an empty request. A different guard filter did not match. Correct.
- `newestWholeAncestor`: it skipped package entries, entries with args and entries of another fingerprint or module. Correct, but it picks the newest by `Written` time (finding 2).
- `openGreen`: a dirty tree returned nil, so it neither read nor wrote. A failed status or HEAD read returned nil. `stillGreen` rechecked clean and the same tree before a write. Correct.
- `greenRun.subtract`: a whole entry dropped the invocation. A package entry gave the remainder and `carried`. An empty remainder dropped the invocation. A remainder over the command budget, a failed list or a command with no `{targets}` ran whole. Correct.
- `greenRun.recordPass`: it recorded whole when the targets equalled all_targets, or when ran plus carried covered every package. A guard run wrote packages under its `-run` args. A narrowed run wrote a package entry. Correct.
- `runGreenAware`, `reportCarried` and `runListed`: a fully carried tier took no slot and wrote a carried record with exit 0. An empty scope still went through `runWithSlots(…, true, …)`. A run that passed recorded green. Correct.
- `gateKeyRefusal`, `flagShapeRefusal` and `withGateScope`: `card` scope with no `--card` and no targets was refused naming the key. `--tier`, `--upto`, `--all` and `--base` beside `--gate` were refused. Explicit targets replaced the key's scope. `all` set `all`, and `diff` dropped `--card`. Correct.
- `diffBaseGroups` and `computeScope`:
  - Tiers sharing a base were grouped.
  - A tier with neither a green base nor a parent merge base ran whole.
  - A dirty tree took the merge base.
  - `--base` gave every tier its one base.
  - Guards from narrowed groups were unioned, which can only add redundant runs, never miss one. Correct.
- `emptyTierFailure`, `isFilterFlag` and `holdsTierTest`: a filter in the call's own args exempted the run, and a guard run's own `-run` did not. A GOOS-constrained file off its platform did not count. `TestMain` was excluded. A package that ran nothing and held a tier test failed, naming the tier and `lyx config gate`. Correct.
- `Shed.enter`:
  - A holding gate marked the row entered. A false gate appended `skipped`, routed by `OnDone` and read the status again.
  - A chain longer than the producer list failed, and an evaluation error failed with the row left not entered.
  - A `pause_before` on a later row paused before its gate.
  - A `pause_after` on a skipped row paused at the `OnDone` row.
  - `persistRouted` cleared `EnteredRow` on routed and skip writes, so a route back to the same row re-evaluated.
  - `OnDone == ""` ended the run `done`. Correct.
- `Goto`, `Relaunch` and `RenameProducers`: `Goto` set `EnteredRow` to the target. `Relaunch` with a target cleared it, and with no target or a target already in effect it left the status unchanged. A rename updated `EnteredRow`. Correct.
- `gotoAdmitted`, `episodeStuckCount`, `activity.go`, `loomengine/coherence.go`, `loomcli/checklist.go:789` and `reportengine/timingspans.go:209` handle `skipped`. Correct.
- `rerouteCount` and `battenshed.countDone` do not handle `skipped`. Findings 4 and 5.

**Findings**

1. **LOW, design.** `internal/gatecli/run.go` (the `openGreen(gateslot.WaitDir(location.AnchorPath()), …)` line) and `internal/lyxcwd/lyxcwd.go:54`.
   - The record directory is built from `AnchorPath()`, which is the worktree path plus `AnchorRel`. The run log directory it replaced used `WorktreePath()`.
   - A `lyx gate test` run from an anchored subdirectory therefore reads and writes a different `green.json`, and creates a `.lyx` directory under that subdirectory.
   - The result is a missed carry. It can also leave an untracked `.lyx` there if the exclude pattern is root-only, which I did not check.
   - D4 says the record is per worktree. `WorktreePath()` is the matching base.

2. **LOW, design.** `internal/gatecli/green.go`, `newestWholeAncestor`. It ranks candidates by `Written` time, not by position in history. Re-running a tier on an older checkout makes that older commit the newest "ancestor", so the `diff` base is wider than D2's newest ancestor. This is sound but loses narrowing.

3. **NIT, design.** `internal/gatecli/green.go:140-155`, the pruning in `with`. The 20-commit budget counts any whole entry, including ones with non-empty args such as `-- -run X`. Many such runs can push out the empty-args entries that D2 needs as bases.

4. **LOW, consistency.** `internal/shedengine/reroute.go:21`, `rerouteCount`. It treats only `Done` as ending the reroute count. D8 says every history reader treats `skipped` as a row passed through. A row gated off, then re-entered later, keeps its old reroute count. The impact is limited to rows that both reroute and carry an entry gate.

5. **LOW, consistency, uncertain.** `internal/battenshed/classify.go:216`, `countDone`. It counts only `Done` entries per row. A batten row skipped by its entry gate (Open-Window, Seed-IDE, Open-IDE) now writes `skipped` instead of `done`. If the caller treats the count as "row completed", a skipped row reads as never run. I did not read the caller to settle it.

I raised no BLOCKING finding.

**Tool calls (4 of 6)**
1. `grep` and `sed` over `internal/gatecli/tiers.go` and `tierrun.go`: the manifest lacked `planTierRuns`, `isTestName` and `guardStepRun`.
2. `grep` for `OutcomeSkipped` and `WaitDir`/`AnchorPath` definitions: the manifest lacked the history readers and the record directory base.
3. `grep` for `Outcome ==/!= Done|Stuck` readers: to find history readers that miss `skipped`.
4. `sed` of `reroute.go`, `classify.go` and `overview.go`: to judge those readers.
