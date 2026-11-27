```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1–F7).

**Findings**

1. **MEDIUM** — `internal/gatecli/cli.go` help text: "--guards, and a narrower scope while … is go, also run the //lym:guard tests the tiers leave out."
   - This is false for the computed and `--base` scopes. `targetsFor` collects `Targets.Guards` only from packages outside the impacted set (`internal/impactset/targets.go:26`, `:133-151`), whatever the guard's tier.
   - So a tier1 guard in an impacted package is skipped by a tier0 run and is not run by the guard step either.
   - It is also inexact for explicit targets with `--guards`, where `scanGuards` sees no inside package and runs every guard, including those the tiers did run.
   - F1's proposal changed only the `--card` path. The old "outside the scope" wording was true for the diff scopes.
   - A fix within the 2000-byte cap: "…tests outside the scope, and under --card those its tiers skip."

2. **LOW** — `internal/gatecli/carddependents_integration_test.go`: the test was renamed from `TestCardDependents` to `TestCardDependents_UnlistableModuleKeepsTheCardsPackagesAndSaysSo`.
   - F6 asked only to delete the success row.
   - `_lym/landing/summary.md`, under "Tests", still names `TestCardDependents` and says it covers the importer being added.

**Checked, no finding**

- **F1 tier rule:** `GuardTest.Tier` is the lowest `Require` tier in the file, so a mixed-tier file could still drop a higher-tier guard. A scan of the marked tests in `internal` and `cmd` found none whose own tier differs from its file's.
- **F1 side effect:** a hub that does not declare a dependent guard's tier now has its card gate refused. That matches how guards outside the scope were already treated.
- **F2, F3, F4, F5, F7:** I built no breaking case.
- **F6 deletion:** the success path is still pinned end to end by the `--card` row. It is now covered only through a spawned process, which the coverage maps may not see.

**Note for the fixer-report:** F1's proposal also asks to update the change description. `_lym/landing/summary.md` still says the guard step "work[s] from the widened set", and that a failed listing only "logs the cause". Report that part as not done.

**Tool calls**

- One call: grep of `internal/impactset/targets.go` for guard handling, plus a scan of the guard-marked test files for per-test versus file tier. My context had neither `targets.go` nor the per-test tiers.
