```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the diff `ed98ccf71..HEAD`. I read the non-test diffs shown above plus the `tiertags`, `tierpurity` and `tmuxisolation` test diffs.

## Per function or branch: case run, result

- `firstRetiredTierSpelling` (`cmd/lyx/tiertags_test.go`):
  - `//go:build tier0 || integration`: found, correct.
  - `//go:build integration && linux`: found, correct.
  - `//go:build !tmux`: found, correct.
  - `//go:build linux && integration`, `tier1 && integration` and `!tier1 && tmux`: **not found, wrong.** See the finding below.
- `admittedTiers` (`cmd/lyx/tiertags_test.go`): `tmux && !windows` now takes the `platformTerms` branch, so it admits several tiers. That does not matter, because the retired check runs first in `tierTagFailure`. Correct.
- `tierTagFailure` (`cmd/lyx/tiertags_test.go`):
  - A file with no test function and `!tmux` is refused, as the fixture row `NegatedRetiredSpellingWithoutTest` expects. Correct.
  - A nil constraint gives `""` from `firstRetiredTierSpelling`, so it falls through to the old logic. Correct.
- `knownTierTags` and `isTmuxTier` (`cmd/lyx/tierpurity_test.go`): `tier0 || tier1` is untagged, `tier2 || tier3` is tmux-tier. Correct.
- `isolationTags` and `isolationTagSets` (`cmd/lyx/tmuxisolation_test.go`): six sets now, and the fixture counts 6, 4 and 2 follow. A file with no `TestMain` fails in 6 sets, and a tier1-only main leaves 4 uncovered. Correct.
- `canCompileWithoutLLM` (`cmd/lyx/llmtier_test.go`): `tier3` alone yields false and `tier0` yields true. Correct. The `//go:build llm` fixture row still yields 1 failure because `llm` is now an unknown tag, so it counts as true. Correct.
- `fileTier` (`cmd/testtiming/spawnscan.go`): `tier2 && linux` gives tmux and tagged, `tier1` and `tier3` give tagged, `tier0` or no line gives neither. Correct.
- `defaultRedundancyTags` (`cmd/testtiming/redundancy.go`): `tier0,tier1,tier2`. Correct.
- `namesGateOnlyTier` (`internal/planparser/verifymodule.go`): `-tags tier2` and `"tier1,tier3"` are refused, `-tags tmux` passes. Correct for the decision.
- `rawGoCommandPattern` (`internal/shellguard/rows.go`): the alternation `(tier2|tier3)` matches `-tags=tier2,x` through `[^[:space:];&|]*`. Correct.
- `listPackages`, `ModuleGuardTests` (`internal/impactset`) and `goListLoader.load` (`internal/planglyph/typesload.go`): the tag strings are `tier0,tier1,tier2,tier3`. Correct.
- `FileTier` (`internal/impactset/guards.go`): `tierN` alone gives N, no tag gives 0. Correct.
- `allow_tmux.go` and the glyphchain `tagged.go`: constraints are `tier2 || tier3` and `tier1`. Correct.
- Hub `gate.yaml` (`/home/knatte/Code/loomyard-LYXHUB/_board/_lyx/config/gate.yaml`): commands are single-tag. Correct.

## Findings

**MEDIUM, design** — `cmd/lyx/tiertags_test.go`, `firstRetiredTierSpelling` (`Eval` closure returns `false` for every tag).
- `constraint.Expr.Eval` short-circuits `&&`, so a retired spelling to the right of a term that evaluates false is never queried.
- Examples that pass the scan with no failure, though the decision says the scan refuses an old spelling outright:
  - `//go:build tier1 && integration`
  - `//go:build linux && integration`
  - `//go:build !tier1 && tmux`
- `//go:build tier1 && integration` also escapes the retired check and lets `admittedTiers` read `integration` as a platform term. The file then reads as `tier1` alone and compiles under `tier1` only if `integration` is set, so it is a silent test loss.
- Evidence: the `Eval` closure at `cmd/lyx/tiertags_test.go` (diff hunk `@@ -70,25 +70,19 @@`), and the fixture rows, which cover only `||`, leading and negated forms.
- Fix: collect the tags with a walk over the expression tree, or have the closure return `true` and scan both polarities, instead of relying on `Eval`.

## Tool calls
1. `git diff ed98ccf71..HEAD -- cmd/lyx/tiertags_test.go cmd/lyx/tierpurity_test.go cmd/lyx/tmuxisolation_test.go`: my context only held the persisted, truncated test diff and lacked those hunks.
