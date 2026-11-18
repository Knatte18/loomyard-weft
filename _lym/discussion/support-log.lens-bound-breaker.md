```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guarantee and claim about existing code in the decision record. Seven claims broke.

**Findings**

1. **BLOCKING. D3, D9 and the batch listing in D12: the reach step's fresh map is never written on a dirty tree.**
   - D3 says the rebuild is the existing `--map` measurement. That measurement writes a map only when no file the package owns is modified or untracked. `internal/gatecli/testmap.go` refuses with "modified or untracked file …; way forward: commit or revert it". `testmaptree.go` reads the uncommitted and untracked files afresh.
   - D2 diffs against "the worktree's files", so a gate on uncommitted edits is the case it targets. That case writes no fresh map for any changed package.
   - Counterexample: a tapestry writer runs its `batch` gate before committing. Every changed package is listed under `unmeasured`. The note then rests on hub maps that lack every new function and block.
   - D9 then reports every changed block as uncovered and plants no mutant.
   - The record never says which sites run on a clean tree. It also never says what the step does when the map write is refused.

2. **MEDIUM. D5 and D10: matching on changed functions alone lets a stale "unreached" note match after the writer adds the missing test.**
   - D2 leaves test files out of the changed functions. D5 and D10 match a report against HEAD by "HEAD's changed functions and their text hashes".
   - Counterexample: the gate prints "`Foo` reached by no test". The writer adds `TestFoo` in a new commit and changes no production function. The changed-function set and its hashes are unchanged, so the old report still "matches HEAD". The review reads `Foo` as unreached, and survivors that `TestFoo` now kills are still listed.
   - This is the workflow D3 exists for. The redundancy report's match also hashes the tests (`TestHashes`), so this match needs the packages' test files, or the maps it read, in its key too.

3. **MEDIUM. D3 says the redundancy step reuses the maps the reach step refreshed, but that changes the redundancy step's semantics.**
   - Today `internal/gatecli/redundancy.go` reads only the hub's shared maps (`testcover.Dir(fabricengine.BoltDir(...))`). It judges the diff's new tests against kept tests that a map already holds (`JudgeAgainst`).
   - A fresh worktree map built at HEAD already holds those new tests. Each new test would be judged against a map that contains itself and its siblings, so contained and unique verdicts shift.
   - The record states the reuse as free and never names this change to the redundancy step's input.

4. **MEDIUM. D7: a triage line inside the failure report can break the parsers that re-read the report.**
   - `gateslot.FailedTests` and `TierRunFailures` (`internal/gateslot/runlog.go`) parse the block under `FailureReportHeader`.
   - A line with two-space indent that is not a `<package> <Test>:` line makes `FailedTests` return ok=false. `TierRunFailures` does the same for a run header that is not a tier.
   - These feed `gateslot.FailedTestsCall`, the next arrival's failed-tests-only rerun, and Finalize's tier-only rerun with its flaky record. A triage line placed at the wrong indent silently turns both off.
   - Separately, the report is capped at 80 lines and the triage lines add more. Once the cut line appears, `FailedTests` treats the report as cut. A successful lookup can therefore push real failures below the cut.
   - The acceptance criterion only says that a *failed* lookup leaves the report unchanged.

5. **MEDIUM. D8 looks at the wrong branch.**
   - D8 takes "the first-parent commits of the current branch after `<rev>`". The caller that meets a repeated failure that is not its own is a run worktree, whose current branch is the run's branch.
   - On that branch, the parent's landings since the last green arrive as the second parent of the merge that brings the parent in. A first-parent walk of the run branch lists the run's own commits and that merge, never the landings.
   - Counterexample: in a run that merged in its parent, `lym gate reach --test X --since <green>` names no landing whose changes X reaches.
   - The verb needs to walk the parent branch, read from fabric's record, or take the branch explicitly.

6. **LOW. D1: "the worktree's map wins, since it is built at the worktree's HEAD" (Auto-mode assumptions) does not hold over time.**
   - A worktree's own map stays under its anchor after HEAD moves. It can also be older than a hub map the prime rebuilt since.
   - Counterexample: a long-lived worktree mapped `pkg` three days ago. The prime remaps `pkg` today, and the reader still prefers the older worktree map.
   - "Wins" should compare the maps' commits or file hashes, not their location.

7. **LOW. D1: the index stamp of size plus mtime can miss a rewrite.**
   - A map rewritten with the same size within one mtime tick, on a coarse-mtime filesystem, keeps its stamp. The index then serves stale pairs.
   - It is advisory and bounded, but the record calls the stamp sufficient without saying so.

8. **LOW. D3: a reach step run from the prime writes the hub's shared maps, not the worktree's own.**
   - Off the prime, `--map` writes the worktree's maps. From the prime, `testMapLocation` and `coverageMapsDir` write the hub's shared maps.
   - A gate key listing `reach` run in the prime (`webster` or `adopt`) would therefore rewrite shared maps from a gate, which test-speed's background rules out. It would also leave the index stale.
   - D3 says "off the prime" but does not say what the step does in the prime.

**Held**

- D12: template defaults reach a hub whose `gates` entries lack `reports`. Gateslot fills a template key that a tuned entry lacks into that entry (`internal/gateslot/doc.go`).
- D10's config refusal and D12's new report names: today's `decodeGates` refuses any report but `redundancy`, so both need the config check changed, as D12 says.
- D6: it writes nothing and takes no slot. Consistent.
- D1: the index is written only by the prime's `--map` run. Held, with D1's own in-memory fallback for an absent or stale index.
- D9: a timeout or memory cap counts as unjudged. Matches the existing mutant judging.
- D11: a board note, not a task. Allowed.
- The PATTERN line lengths are under the 280-character cap.

**Tool calls**

- Read the failure-report parsers in `internal/gateslot/runlog.go`. Context only showed that the header was referenced there.
- Grepped `gateslot/config.go` for report validation and `gatecli` for the map directories. I needed whether the redundancy step reads hub or worktree maps.
- Grepped `websterengine` for the fixer report path and `gatecli/testmap*.go` for the dirty-file guard. Context held the doc's claim but not where the refusal is.
- Read `decodeGates` and the gateslot doc lines on filling template keys into tuned entries. I needed whether hub keys without `reports` get template defaults.
