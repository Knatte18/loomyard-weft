```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Task: run each changed function on concrete inputs and report wrong results.**

**Per function or branch: case run, verdict**

- `tiergate.Selected`
  - `""` gives {0,0}: correct.
  - `"1"`, `"0-2"` and `" 1 "` parse correctly.
  - `"-1"` errors because the empty low text fails `Atoi`; `"1-"` errors the same way.
  - `"2-0"` errors on `high<low`; `"1-2-3"` errors because `Cut` leaves `"2-3"` as the high text.
  - `"+1"` and `"01"` are accepted because `Atoi` allows them (NIT, below).
- `tiergate.Require`
  - On a parse error it calls `t.Fatal`, so a malformed variable fails every test instead of skipping silently: correct.
  - For `Holds(tier)` false it skips: correct.
  - `Benchmark` and `Fuzz` get a `testing.TB`: correct.
- `Selection.Holds` and `Selection.String` (boundary Low==High, range): correct.
- `impactset.RequiredTier`
  - It resolves an aliased import name and returns the tier from a `Require(t, N)` first statement.
  - A body with no statements, a nil body, or a first statement that is not a call returns `false`: correct.
  - A non-literal or negative tier returns `false`, so the tier guard fails the file rather than hiding it: correct.
  - The first argument is not checked, so `Require(x, 0)` passes; this is harmless.
- `impactset.FileTier`
  - It takes the lowest `Require` tier over all functions, including helpers whose first statement is a `Require` call.
  - A file with no `Require` falls back to the tag, and `max(tier,0)` returns 0 when there is no constraint or no tier tag: correct.
  - Run on an old-tree file tagged `tier1 || tier2`, it returns 1: correct.
  - Run on a file with a `TestMain` and a helper and no `Require`, it reads tier 0. A support file that was tier1-tagged used to read tier1. See finding 2.
- `guardNamesInFile`: `tier>1` returns `GuardWrongTier`. A `//lym:guard` in a tier0 or tier1 file is accepted: correct.
- `ModuleGuardTests`: `go list -tags tier0..3` still works on untagged files: correct.
- `testtiming.tierSelection`
  - `"tier0"` gives `"0"`; `"tier1"` gives `"1"`; `"tier1,tier2"` gives `"1-2"`.
  - `"tier0,tier2"` is refused with a message naming the missing tier: correct.
  - Duplicates are collapsed; a tag naming no tier is ignored.
  - `"tier-1"` gives `Atoi("-1")`, which appends -1 and returns `"-1"`, which the selector then rejects at run time (NIT, below).
- `testtiming.main` flow
  - Redundancy defaults of `tier0,tier1,tier2` give `0-2`: correct.
  - An error from `tierSelection` skips the run, removes the temp directory and exits 1: correct.
  - `os.Setenv` happens before any child starts.
  - The children take `os.Environ()` (`redundancy.go:633`, `resources.go:132`), so they inherit the variable: correct.
  - For `run()`, `prebuildLym` takes the `base` env. `withGateSlot` builds it from `os.Environ()`, so the variable reaches it as well.
- `scanSpawns` and `loadPkgSource`
  - `tagged = tier>0` and `tmuxTier = tier==2` come from `FileTier`: correct for test functions.
  - A `TestMain` in a support file now reads untagged and gets an empty verdict. No real test is named that, so this is harmless.
  - A tier-1 test that calls a helper in a support file still has the helper followed through `analyze`: correct.
- `tmuxkit.tmuxAllowed` and `checkSockets`
  - `""` and `"1"` give false.
  - `"2"`, `"3"`, `"0-2"` and `"1-3"` give true.
  - A malformed value gives false; every test of that run fails anyway.
  - This matches the old tag behaviour for `-tags tier1` runs and for runs that name tier2 or tier3.
- `tierFailure` (`cmd/lym/tiertags_test.go`)
  - A nil constraint gives `""` from `firstTierTerm`: correct.
  - A tier tag in any position, negated or not, is flagged.
  - A `Require` that is not the first statement is flagged.
  - An `Example`, a mixed-tier file or a missing `Require` is flagged.
  - `TestMain` and a function literal in a string literal are not flagged: correct.
- `TestTierPurity_UntaggedTestsSpawnNothing`
  - A file with no test function returns early, so a support file is never scanned. See finding 1.
  - A tier>0 file takes the tmux-reference check and returns; a tier0 file takes the banned-token, gitkit, tmuxkit and sleep checks: correct.
- `llmTierFailures`
  - A support file that imports `llmkit` passes, a tier3 test passes, a tier1, tier2 or `Require`-less test fails, and the `LookPath("claude")` check is independent of tier.
  - The fixture rows confirm each (`cmd/lym/llmtier_test.go:196-207`).
  - The guard cannot see a tier0 test that calls an `llmkit` helper declared in a support file; the doc comment admits it.

**Findings**

1. **LOW, design**, `cmd/lym/tierpurity_test.go:97-99`: support files are skipped (`len(testFunctions(file))==0`), so a helper file that used to be an untagged tier0 file and contained `exec.Command`, `gitexec.Run` or `hubforge.NewHub` is no longer caught. The old guard scanned it; the summary states the weakening and the pattern doc states the bound. Judge whether the loss is accepted. Evidence: `cmd/lym/tierpurity_test.go:3` and `97-99`.
2. **LOW, design**, `internal/battencli/testmain_integration_test.go` against the removed `testmain_test.go` at the base: the removed tier0 `TestMain` replaced `spawnPairPush` with a refusing stub ("unit tests push no pair"). The remaining one installs the real `fabricengine.PushPairAnchored` and runs in every tier's binary. A tier0 test that reaches `wire.go:980` or `wire.go:1425` would now push for real instead of failing loudly. No tier0 test reaches it today as far as I could see, but the safety net is gone. Evidence: the `TestMain` bodies as compared in the manifest, and `internal/battencli/wire.go:62-63`.
3. **LOW, design**, `hubreconcile` and `logger`, `testmain_integration_test.go`: the remaining `TestMain` now also runs `HermeticGitEnv` and, in `hubreconcile`, sets `FABRIC_SKIP_PUSH=1` for tier0 runs, where the removed companion did neither. This is benign and only widens the environment, but it is a behaviour change of tier0 runs.
4. **NIT**, `internal/testkit/tiergate/tiergate.go:53-55`: `Atoi` accepts `"+1"` and `"01"`, so `Selection.String` does not round-trip such a value. No correctness impact.
5. **NIT**, `cmd/testtiming/main.go:191`: `-tags tier-1` produces `"-1"`, which fails later at run time in the selector instead of being refused up front.
6. **NIT**, `internal/testkit/tmuxkit/tmuxkit.go:18`, `10`: the doc still says "either tag" and "tagged test", but tmuxkit now reads the selection. A stale comment, not a code error.

No wrong result, swallowed error or inconsistent failure-path state found in the selector, in the static readers or in the `testtiming` env flow.

**Tool calls**
1. Read `cmd/lym/llmtier_test.go` from line 125: the manifest held only a partial diff of the new LLM guard walk.
2. Grep for `os.Environ`, `HolderEnv` and `prebuildLym` in `cmd/testtiming`: to confirm that `LYM_TEST_TIER` set by `main` reaches the go child commands.
