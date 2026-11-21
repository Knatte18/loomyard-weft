```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F3, F5–F14, and F4 as amended by the ruling).

**1. MEDIUM — F10, `conflictingResultsWord` in `internal/gatecli/trx.go`**
- The summary is that `-r` is refused as if it were the results directory. In `dotnet test`, `-r` is the short form of `--runtime`, and `--results-directory` has no short alias.
- A call such as `lym gate test --tier 0 -- -r linux-x64` is now refused as holding a results directory. The reviewer's fix named only `--results-directory`, and the fixer's `case "--results-directory", "-r":` widened it wrongly.
- The fix is to drop `-r` from the case; `-l` for `--logger` is a real alias and can stay.
- Evidence: the `case` line in `conflictingResultsWord` and the `{args}` pass-through, from my knowledge of the `dotnet test` option list. I made no tool call to confirm it, so it carries that uncertainty.
- The tests in `tierrun_spawn_test.go` only cover `--results-directory` and `--logger trx;…`, so nothing catches this.

**2. LOW — F10, where the refusal happens**
- The refusal sits in `runInvocations`, after the tier's slots are held and after the shared build step has run (`run.go:624` runs the build before the tier command). A bad call word therefore costs a full build and slot wait before it is refused.
- `planTierRuns` already refuses words after `--` for a tier with no `{args}` (`tiers.go:116`). The same refusal could go there for a tier with `results: trx`.
- The review proposed `runInvocations`, so this is a cost, not a wrong result.

**3. LOW — F14, the env line prints once per attempt, not once per call**
- `runTierChecked` is called once per attempt, and `retryTier` calls the attempt twice on a shortage or crash rerun. The line therefore prints to stderr twice on a rerun.
- The rerun's log also carries it, after the rerun announcement.
- The NIT's stated goal was a single line per call. It is met for `--copies`, but not for a rerun.
- Evidence: `retryTier` in `tierrun.go`, which calls `attempt()` twice, and `run.go:631`, where the attempt wraps `runTierChecked`.

**4. LOW — F4 with the ruling, `MeasureCall` refusing a tier without `{args}`**
- The darn table build now fails for any hub whose `darn-write` tier command lacks `{args}`. That includes the previously documented recipe, which this fix repaired in `docs/dotnet-hub.md`.
- The `gate.yaml` guidance in `docs/overview.md` and the template comments do not say a `darn-write` tier needs `{args}`. A hub can hit the refusal with no earlier warning.
- The refusal text does name the config fix through `useRefusal`, so this is a stale reference, not a wrong behaviour.

No finding on F3, F5, F6, F7, F8, F9, F11, F12 or F13. I considered F12's mid-run `Flush` and F11's `Error` filter and found no concrete failing case.

Tool calls: none; my context held every file the lens needed.
