```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's changed code and tests (lens fix-code-correctness).

No findings.

I traced each changed branch by hand against the code in my context and the green gate (718 passed, 0 failed):

- **`newArrasRun` signature change:** the package compiled and passed the gate, so no caller was left on the old signature. `shuttle.RunFn` is captured as `dispatch` before `designWrongBouncerJudge(env)` replaces it, so the scoped wrapper falls back to the original dispatcher for Discussion's judges. `split.HardMaxBounces` is set before `env.Segments[splitPerchSegment] = split`, so the hard cap takes effect.
- **Goto over a settled perch:** the test fails if the goto does not re-arm the perch. Without the clear, the Bouncer would replay the settled CONVERGED verdict, and the history would lose the second `Raddle-Burler-Review` entry. The expected list has the goto's own `Raddle-Write` entry, which matches what `shedengine.Goto` appends and what the gate showed.
- **Goto while the perch awaits its parent:** the test fails on any fresh cycle. A clear would add review entries, so the review count would differ, and a halt other than circling would fail the cause check. Budget 1 under hard cap 4 gives a circling escalation below the cap, which the gate confirmed.
- **`ResolveJudge` split rows:** "split_judge alone" gives seed and judge `opus`, since the seed falls back to the resolved judge. "split_judge with seed" gives `haiku` and `opus`, since `seed` is set. Ignoring `SplitJudge` would make the `Contains` checks fail.
- **`ResolveReview` rows:**
  - The empty-raddle case holds because `Split` equals `Models` and `SplitKeys.Fix` is `fixer_model`.
  - The raddle-set case fails if the fallback is removed, because the split fixer would resolve to the `claude` engine instead of `other`, and the key would not be `raddle`.
- **Added config, subdir and profile rows:** each asserts a value that differs from what the old code would give. The `Raddle-Bouncer` → `split` row, the negative `split_*` refusal rows and the arras ladder per profile (`discussion` rounds, `split` rounds, hard cap one above) all passed.
- **Docs additions:** they agree with `config.go`. The `split_*` keys exist, `split_auto_accept` is a guard key defaulting to false, the fixer model falls back to `raddle`, the judge has no seed key, and the two fan keys are closed to the local layer.

One weak assertion, not a finding: in the settled-perch test, the final `round-1-review.md` stat passes whether or not the clear happened, because the fresh cycle writes that file again. The history list is what discriminates, so the test still catches the defect.

Further tool calls: none. My context held every changed file and the gate output.
