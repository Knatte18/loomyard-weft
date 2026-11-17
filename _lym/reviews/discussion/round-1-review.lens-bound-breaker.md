```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I tried to break every bound, guarantee and claim about existing code in the decision record. Four claims broke; the rest held.

**Claims attacked**
- D1 Bound "the refused flags are the only gate features map mode skips": **broken**.
- D1 detached keying: `--map` among the words `testRequest.words` hashes, so a map call never joins a plain run: **held**, provided the flag is added to `words()`. The fixed word list makes that a plan step.
- D1, card fixer reachable from a red map call: **held**. `handsOff` needs `--gate`, which `--map` refuses.
- D2 one-ahead build, plus cancel-and-wait on an early end: **held**, since `takeBuild` already cancels and waits.
- D3 cover set, tier env and `lym` built once: **held**. The claims match `startBuild`, `runTest` and `buildLym`.
- D4 "count at most `memory_mb` divided by the per-test cap": **broken** when `memory_mb` is below the per-test cap.
- D4 "the tier's cap bounds the run's test processes together": **broken** when the two tiers run at once.
- D4 headroom clamp and memory guard "as today": **held**.
- D5 "the list makes each collision visible": **broken** past the call's own output.
- D6 "a map is written only from committed files", checked "before a package's coverage build": **broken**.
- D6 "any file under the package's directory": **broken** for a package with subpackages.
- D7 slot hold and the `slots: all` bound: **held**. `slots: all` is `Tier.Slots == 0`, so "larger" has to read 0 as every slot.
- D7 no green record, so no later skip: **held**.
- D7 failed build keeps its old entries "as today": **held** (`writeFailed`).
- D8 "each write replaces the map at once", last write wins: **held**. `WriteMap` goes through `state.WriteJSON` under the map's `.lock`; I did not read the atomic-rename step itself.
- D8 old `.build.lock` files inert: **held**. The map's lock is `<map>.lock`, a different file.
- Constraint `PATTERN-gate-slot` `-trimpath`: **held** as an instruction. Today's holder gets it from `HolderEnv`, and a tier run gets it from the tier's `GOFLAGS`.
- Build-cache fact: not attacked. It cites a study, not code.

**Findings**

1. **MEDIUM · design · D6 (decision-record.md:96-105).** The pre-build dirty check over linked packages cannot run before the build, and an edit made and reverted mid-run still reaches a map.
   - The linked set comes from the build itself: `startBuild` runs `LinkedPackages` and then `BuildBinary` in one goroutine (testmappackage.go:224-236).
   - The one-ahead build of D2 starts while the previous package's tests run (testmappackage.go:187-188).
   - Today's code checks linked packages only once the build is taken up (testmappackage.go:172).
   - Counterexample: a linked file is edited while `go test -c` compiles, then reverted before the next status read. The binary instruments the edited code, and the map stamps HEAD's hashes on it.
   - Today's doc states this as a bound (internal/gatecli/doc.go:1133-1134). The record drops that bound and claims committed files only.
   - Fix: D6 states the check order the build forces (list, build, then check), and keeps the edit-and-revert bound.

2. **MEDIUM · design · D4 (decision-record.md:75-79).** The `memory_mb` quotient has no floor.
   - The template's `redundancy_test_memory_mb` is 2048 (internal/gateslot/template.yaml:7), and `memory_mb` accepts any value from 1.
   - Counterexample: a tier with `memory_mb: 1024` gets a count of 1024/2048 = 0. The record does not say whether the run then measures nothing, hangs, refuses, or floors at 1.
   - A floor of 1 would break the claim at decision-record.md:79 that the tier's cap bounds the processes together, since one 2048 MB process exceeds 1024.

3. **MEDIUM · design · D4 (decision-record.md:75-79, 82-83).** The record never says whether a package's tier 0 and tier 1 tests run at once.
   - D4 bounds each tier separately: "a tier's tests run side by side up to that tier's `parallel`".
   - Counterexample on the live hub config (tier0 and tier1 both `parallel: 4`): if both tiers run together, 8 processes × 2048 MB = 16 GB. Each tier's `memory_mb / cap` clamp holds, but neither tier's `memory_mb` bounds the total.
   - That breaks "the same total as a scope around them".

4. **MEDIUM · design · D1 Bound (decision-record.md:41, 49).** Several flags of `lym gate test` are neither refused nor said to apply.
   - The flags: `--stop-on-fail`, `--dir`/`-C`, `--main-packages` and `--tags` (internal/gatecli/cli.go:119-130).
   - Counterexample: `lym gate test --map --stop-on-fail ./internal/x`. The record does not say whether the run ends at the first failed test, the first failed package, or not at all.
   - Likewise `-C <dir>`: the tier run directory means nothing when every test runs in its package directory.
   - The Bound's claim to list every skipped feature is false as written.
   - The attack-surface rule wants each such flag refused or bounded.

5. **LOW · design · D5 (decision-record.md:89-94) with D7 (decision-record.md:110).** The list of tests that passed only alone exists only in the call's output and log.
   - D7 writes no run record, and nothing in the map marks such an entry.
   - Counterexample: a genuinely flaky test fails beside others, passes alone, and enters the map as a plain pass with full coverage. `lym gate redundant` and the report step then count it as a cover.
   - A detached agent call's list is visible only in that call's replayed output.
   - The bound "the list makes each collision visible so its test can be fixed" lasts only as long as that one output is read.

6. **LOW · consistency · D6 (decision-record.md:99) and criterion (decision-record.md:183).** "Any file under the package's directory" also takes in its subpackages' files.
   - Today a package owns files only down to the next package directory or nested module (internal/gatecli/doc.go:1071-1072).
   - Counterexample: a dirty file in `internal/x/sub/` marks `internal/x` unmeasured, although its own files are clean.
   - The same holds for the module root package, if one exists: any dirty file anywhere would block its map.

**Tool calls**
1. grep for `cappedParallel` and `WriteMap`: to read the memory clamp and test D8's write claim; neither was in context.
2. Read the hub's `gate.yaml` tiers and grep `Slots` in config.go: to see the live tier parallels and `memory_mb`, for D4 and D7.
3. Read the config.go span and grep `cappedParallel`: to find where the clamp is defined.
4. Read the `cappedParallel` span in redundant.go and grep `Slots` in tiers.go: this found `Slots == 0` means every slot, for D7.
5. grep for the `cappedParallel` method: no output.
6. Located `cappedParallel` as a field set to `memoryCappedParallel`. I could not read that function within the tool budget, so whether it floors at 1 is unverified; finding 2 does not rest on it, because D4's `memory_mb` quotient is new.
