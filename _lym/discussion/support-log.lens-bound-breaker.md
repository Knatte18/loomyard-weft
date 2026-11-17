```yaml
lens: bound-breaker
outcome: reported
```

The bound-breaker lens found one BLOCKING, two MEDIUM and three LOW findings.

**BLOCKING — D6, a clean tree at start does not keep the commit stamp true.**
- **Summary:** D6 drops the dirty-file check the verb runs before each package. A file edited mid-run gets a map stamped with HEAD's hashes but built from the edited working tree.
- **Evidence:** `internal/gatecli/testmappackage.go` says it "discards the runs when a linked package's files are dirty or HEAD moved under its tests". `dirtyOwnedFile` runs per job. D6 checks dirtiness only at the start and HEAD before each write.
- **Counterexample:** an agent runs `lym gate test --map --all` detached in a worktree. While it runs, someone edits `internal/x/y.go` without committing. A later package that links `internal/x` builds from the edited file, and its map is written with HEAD's commit and hashes. `lym gate redundant` then trusts a map that matches no commit.
- **Fix:** either re-check the package's linked trees for dirty Go files before each build and before each write, or state the bound in D6.

**MEDIUM — D1 and D7, a detached call can lose `--map`.**
- **Summary:** "Agent calls go through the detached runner like any other gate call" holds only if `--map` enters the call's key words.
- **Evidence:** `testRequest.words` in `internal/gatecli/detachfront.go` is a hand-built flag list: `dir`, `gate`, `tier`, `upto`, `card`, `main-packages`, `base`, `all` and so on. `detachKeyDir` keys the run directory on those words.
- **Counterexample:** a new flag left out of that list makes `lym gate test --map ./pkg` and `lym gate test ./pkg` share one key. Over an unchanged tree the second call joins the first's run and gets a tier result as a map result, or the reverse. If the runner replays the call from those words, it drops `--map` and runs plain tiers.
- **Fix:** D1 should require `--map` to be part of the call's key words.

**MEDIUM — D7, the slot hold has no stated bound.**
- **Summary:** holding the larger of tier 0's and tier 1's slot asks for the whole call can hold every slot of the hub for the whole build.
- **Evidence:** `internal/gateslot/tiers.go` accepts `slots: all`. The study puts a full build at about 16 CPU-minutes, around 4 minutes wall time at four side by side, longer at lower parallelism.
- **Counterexample:** on a hub whose tier 1 asks `slots: all`, `lym gate test --map --all` holds every slot for minutes. Every card, batch and landing gate on the hub waits that long.
- **Fix:** state this as the bound, or hold the slots per tier or per package.

**LOW — D4 and D5 disagree about a test the guard kills.**
- **Summary:** it is not said whether such a test is rerun alone.
- **Evidence:** D5 reruns a test that "fails or times out" beside others. Commit 580b27a1 records a guard-killed testmap entry as failed and never reruns it. The D4 acceptance criterion says "failed entry naming the kill", which suggests no rerun.
- **Fix:** say so explicitly. Otherwise an implementer may rerun a memory hog alone and double the cost.

**LOW — Constraints, the build-cache fact is overbroad.**
- **Summary:** "A coverage build shares no cached package with the normal gate build" claims more than the study measured.
- **Evidence:** the study shows only that the instrumented set is reused between cover builds. Packages outside the cover set, such as the standard library and external modules, compile the same way and share the cache.
- **Fix:** narrow the claim to the instrumented packages.

**LOW — D3, nobody is named to build `lym` for tier 1.**
- **Summary:** D3 says tier 1 runs with "the gate's prebuilt `lym`", but no step in map mode builds it.
- **Evidence:** per `PATTERN-gate-slot`, the shared build step runs inside the slots of the first tier that opts in, and map mode runs no tier command. Today the verb builds `lym` itself (`buildLym` in `testmappackage.go`).
- **Fix:** say the map run builds `lym` once per call.

Claims attacked:
- D1, refused flags and the `go-json` tier check: held.
- D1, the detached runner: broken (MEDIUM above).
- D2, rebuilding each map whole: held. Old maps and leftover `.build.lock` files stay readable and harmless.
- D3, the spawn-marked and outcome entries: held.
- D3, the prebuilt `lym`: gap (LOW above).
- D4, the clamp to `memory_mb`: held, provided `memory_mb` caps the headroom even on a host with no cgroup limit. A host whose cap cannot cover a process tree already falls back to one test at a time. The builds outside the cap are stated.
- D4, guard coverage: held. `LYM_GATE_TEST_PID` is only set inside an outer tier's environment.
- D5, the alone-rerun: held. The guard-kill interaction is the LOW above.
- D6, untracked Go files: held. `.scratch/` is gitignored (`**/.scratch/`), so files there are never listed as dirty.
- D6, the mid-run edit: broken (BLOCKING above).
- D6, HEAD moving: held.
- D7, no green record: held.
- D7, the slot hold: unbounded (MEDIUM above).
- D8, last write wins: held. Within one worktree a HEAD move ends the slower run, and the prime and other worktrees write different directories.
- D9, `lym gate testmap` becoming an unknown command: likely holds through the gate group's `ArbitraryArgs` handler, which answers with its own error. Not verified.
- Constraints, the build-cache fact: overbroad (LOW above).

Tool calls beyond my context:
1. Lines 50–66 of `internal/gatecli/cli.go`, `.gitignore` and the `LYM_GATE_TEST_PID` sites: to check the gate group's argument handling for D9, whether `.scratch` is ignored for D6, and when the guard is off for D4.
2. The join rule in `internal/gatecli/detachfront.go`, the dirty checks in `testmap.go` and `testmappackage.go`, and the `slots: all` handling in `internal/gateslot/tiers.go`: the record's D1, D6 and D7 claims rest on these and my context did not hold them.
3. The words line of the detached call in `detachfront.go`: to see what the run key is built from.
4. `testRequest.words` in `detachfront.go`: to confirm the key is a hand-built flag list that a new flag must join.
