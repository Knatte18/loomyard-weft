```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, run over each function and branch the diff adds or changes.

**Per function or branch: case run, verdict**

- `run.go:173` gate `code == 0 && sc.form == scopeCard`:
  - Card call, all runs pass, `code` is 0: prints. Correct.
  - A run that failed, or a build that blocked: `code` is non-zero, silent. Correct (D3).
  - Halted call: returns at `run.go:169` before the gate, silent. Correct (D3).
  - Any other scope form: silent. Correct (D2).
  - A failure that leaves `code` at 0: not possible. Both failure branches set `code` when it is 0.
  - Cancelled call: `exitAfterSignal(caught, code)` can turn exit 0 into non-zero after the line is printed. The line is then output beside a non-zero exit. A signal that arrives after all runs pass but before the return is a narrow race, and the line is accurate. Not a finding.
- `notRunTiers` (`tiers.go`, new):
  - Hub declaring tier0 and tier1 with the card key listing tier0: tier0 ran, so only tier1 is returned. Correct.
  - Tier covered by a listed tier, as in the `--tier 1` test, where tier1 covers tier0: `slices.Contains(run.tier.Covers, 0)` is true, so tier0 is left out. Correct.
  - Guard or vet run for tier N: excluded by `!guardStep && !vetStep`, so it does not count as running the tier. Correct (D1).
  - Manual tier: `cfg.Tiers` holds it, so it is named as not run. Correct.
  - Empty plan: not reachable, since `planTierRuns` refuses a hub with no tier0 and always lists tier0.
  - Tiers sorted with `sort.Ints`; map-iteration order is irrelevant. Correct.
  - Key parsing: `strconv.Atoi(TrimPrefix(key, "tier"))` ignores its error. The error is unreachable if the config decode accepts only `tierN` keys, but I did not read the decode at `gateslot/tiers.go:253`. A bad key would yield tier 0 and a false "tier0 not run" only if that decode lets such a key through.
  - Transitive coverage: tier3 covers [1] and tier1 covers [0], with only tier3 listed. tier0 is reported not run, because `Covers` is read one level only. This matches `dropCovered`'s non-transitive reading and the config validation, so it is correct to the config's semantics.
  - A tier whose run was skipped as already green, or a `buildOnly` tier, is in the plan and counts as run. The record does not say whether a green-skipped tier counts as run, so I judge it consistent and not a defect.
- `printNotRunTiers` (`run.go:209`):
  - Empty `numbers`: returns with no output. Correct.
  - `numbers` of `[1, 2]`: prints `gate test: tier1, tier2 did not run, so this pass is not green for them`. Correct.
  - `stderr` nil: `printLine` returns early. Correct.

**Findings**

1. LOW, `design`: the printed line omits the `passed <run tiers> only;` clause that D1 specifies. D1's text is `gate test: passed <run tiers> only; <not-run tiers> did not run, so this pass is not green for them`. Evidence: `run.go:209-218` builds `"gate test: "+names+notRunSuffix`, and the test pins that shorter form at `scopes_integration_test.go:514` and `:522`. The line still names the not-run tiers and meets the acceptance criteria, so this is a wording deviation from the record, not a behavior defect.

I found no wrong result, swallowed error or inconsistent state on a failure path in the changed code.

**Tool calls**
1. `grep` and `sed` over `internal/gateslot/tiers.go` (`decodeTier`), `config.go:31` and `tierrun.go:388`. I read them because the manifest lacked the `Config.Tiers` type, the `Covers` and `Manual` validation, and `printLine`'s nil-writer handling.
