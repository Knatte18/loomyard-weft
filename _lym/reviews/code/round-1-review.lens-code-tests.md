```yaml
lens: code-tests
outcome: reported
```

Task: for each behaviour the diff adds or changes, name the test that pins it, or report none.

**Per behaviour: pinning test**

| Behaviour | Pinning test |
|---|---|
| `tiergate.Selected` parsing: unset, single tier, range, spaces, word, reversed range, negative, open range | `TestSelected_ParsesTheVariable` (`internal/testkit/tiergate/tiergate_test.go:12`) |
| `Require` skips outside the selection and runs inside it | `TestRequire_RunsOnlySelectedTiers` (`tiergate_test.go:43`) |
| `Require` fails the test when `LYM_TEST_TIER` does not parse | none (finding 1) |
| `Selection.String`, used in the skip message | none (finding 2) |
| `tmuxkit.tmuxAllowed` reads the selection, so tier2 or tier3 allows tmux and the rest do not | none (finding 3) |
| `tmuxkit.checkSockets(dir, allowed)` | no changed test row; no `tmuxAllowed` or `LYM_TEST_TIER` reference in `internal/testkit/tmuxkit/*_test.go` |
| `impactset.RequiredTier` / `FileTier`: a `Require` call with the plain import name, an aliased import, and a tier2 file | `TestFindGuardTests` rows at `internal/impactset/guards_test.go:110`, `:115` and `:120` |
| `FileTier` fallback to the tag for an older tree | `TestFindGuardTests` rows at `:65`, `:70`, `:127` and `:134` |
| `RequiredTier` rejects a `Require` call that is not the first statement | `TestTierFailure_FixtureSources` row `RequireNotFirst` (`cmd/lym/tiertags_test.go:149`) |
| `RequiredTier` rejects a non-literal tier or a differently named package | none (finding 4) |
| Tier guard: missing `Require`, two tiers in one file, tier tag, retired spelling, `Example` function, support-only file | `TestTierFailure_FixtureSources` (`cmd/lym/tiertags_test.go:133`) |
| `Benchmark` with `Require` | the `BenchmarkWithRequire` row (`tiertags_test.go:150`) |
| `Fuzz` function with `Require` | none; `holdsTestFunction` handles `Fuzz`, but no fixture row has one |
| Tier-purity guard: tier0 files scanned, tier1 files checked for tmux spawns, support files skipped | `TestTierPurity_UntaggedTestsSpawnNothing` (`cmd/lym/tierpurity_test.go:88`), a scan of the real tree with no fixture rows |
| Tier-purity "support file skipped" and "tier1 file references a tmuxkit spawn" branches | none as a fixture (finding 5) |
| `llmtier` guard rewritten to read `Require` tiers | only the real-tree scan test; I did not inspect its fixture rows |
| `cmd/testtiming` `tierSelection`: one tier, range, unordered tags, no tier tag, a gap | `TestTierSelection` (`cmd/testtiming/main_test.go`) |
| `main` setting `LYM_TEST_TIER` from `-tags` | none (finding 6) |
| `spawnscan` reading the tier through `FileTier` | `TestScanSpawns` (`cmd/testtiming/spawnscan_test.go:130`), rows for the tier2, tier2 && linux, tier1 and tier0 fixtures |
| `spawnscan` with a `Require`-opened file | none (finding 7) |
| `loomshed` `testTierWayForward` text and the `test-tier-unstated` check | `internal/loomshed/gates_test.go:1174-1180`, rows whose intents still state `//go:build tierN` |
| `test-tier-unstated` accepts an Intent line naming a bare tier (`tier0`) | none (finding 8) |
| `tmuxisolation` guard drops its tagged-package floor | `TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain` (real-tree scan) |

**Findings**

1. LOW, design. `Require` with a malformed `LYM_TEST_TIER` calling `t.Fatal` has no test. Location: `internal/testkit/tiergate/tiergate.go:62-67`. Evidence: `tiergate_test.go` never calls `Require` on a bad value. It is the only path that stops a typo from skipping every test silently, which is what the "no test may go silent" decision is about. A test could run `Require` on an inner `testing.TB` fake, or in a subprocess.
2. NIT, design. `Selection.String` is untested. Location: `tiergate.go:35-40`. It only feeds the skip message, so the risk is low.
3. MEDIUM, design. `tmuxAllowed` has no test. Location: `internal/testkit/tmuxkit/tmuxallowed.go:9-12`. The `checkSockets` flip that tier2 and tier3 allow tmux while tier0 and tier1 forbid it is the behaviour moved from the build tag to the selector, and nothing sets `LYM_TEST_TIER` in the tmuxkit tests. If it were inverted, nothing would fail. Pinning it needs `t.Setenv` rows over `"1"`, `"2"`, `"0-2"`, `"3"`, `"junk"` and unset.
4. LOW, design. `RequiredTier` has no row for the shapes it rejects besides "not first" and "no import". Location: `internal/impactset/guards.go:253-276`. Missing rows: a non-integer-literal tier such as `Require(t, tier)`, a wrong package qualifier, and a one-argument call. A regression there would make `FileTier` fall back to the tag path silently.
5. LOW, design. The tier-purity guard's new branches have no fixture rows. Location: `cmd/lym/tierpurity_test.go:97-108`. These are the "no test function means skip" branch and the "tier>0 file with a tmuxkit spawn reference below tier2" branch. Only the real-tree scan covers them, and that scan is green whether or not those branches work. Compare `TestTierFailure_FixtureSources`, which is fixture-driven.
6. LOW, design. `main` setting `LYM_TEST_TIER` is not pinned. Location: `cmd/testtiming/main.go:129-135`. `tierSelection` is covered, but nothing checks that `main` passes `goTestTags(tags)` or that the variable reaches the `go test` child. A wrong or dropped `os.Setenv` would run tier0 only and report green.
7. LOW, design. The `spawnscan` fixtures still use build tags. Location: `cmd/testtiming/spawnscan_test.go:39, 108-113`. They exercise `FileTier`'s tag fallback, not the `Require` path that every real test file now takes. Evidence: `fixtureTier0Tests` and the other fixtures declare `//go:build tierN` and no `Require` call. A fixture with `tiergate.Require(t, 2)` would show a tmux-tier file classified out of process through the new reader.
8. LOW, design. No `test-tier-unstated` row uses the new wording. Location: `internal/loomshed/gates_test.go:1174-1180`. The rows still state `//go:build tier1`. The new way-forward text names a bare `tier0` and the `Require` call, but the gate's matching of an Intent line like "new_test.go is tier0" is not shown to be pinned. I did not read `testtier.go`'s matching code, so this may be covered; verify before keeping it.

**Tool calls (2)**
1. A `git diff` of `cmd/testtiming/main_test.go` plus greps of `tmuxkit` tests, `impactset` `guards_test.go` and `loomshed` tests. My context did not hold the test bodies of those packages.
2. A diff of `internal/loomshed/testtier.go` and greps of the `loomshed` tests, then `spawnscan_test.go` lines 100-175. My context did not hold the fixtures or the `loomshed` test rows.
