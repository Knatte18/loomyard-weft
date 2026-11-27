```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens over the diff's changed functions and branches.

**Functions and branches, with the case I ran:**
- `newScopesHub`: on a fresh copy, the seed commit on the prime and the `--ff-only` merge put the pair at the prime's HEAD, so `mergeBase` equals the pair's HEAD. The plan is written under `_lym/plan`, which the code repo excludes. Correct.
- `scopesHub.writeScoper`, `writeDotnet`, `writeInert`: tried with `mutate` nil and with `mutate` set. Each writes this row's own `configPath`. Correct.
- `scopesHub.edit` after the restore was dropped: tried on an existing file and on a new file in an existing directory (`other/added.go`). There is no `MkdirAll`, as at the base, and every caller's directory exists. Correct.
- `scopesHub.removeGreen`: tried with the record missing. `IsNotExist` is ignored. Correct.
- `scopesHub.wantRefusal`, `wantArgs`, `wantCarried`, `wantDerivation`, `real`, `commitChange`, `requireFree`: these are moved closures, now reading this row's `f`. Correct.
- `noCarryTier1`, `scopesSharedBuild`, `vetSummary`, `wantInertEmpty` at package level: their bodies are unchanged. Correct.
- The scopes rows that now open with a priming `--all`:
  - "a changed tier command, a different tree and -- -count=1 each break the carry": priming is asserted to run `./...`, then the carry is checked. Correct.
  - "an --all run runs after a committed non-inert change although a whole ancestor is green": the priming run records a whole green at the merge base, then a non-inert commit runs whole. Correct.
  - "an --all run on a dirty tree runs whole although the history is inert-only": the priming run records whole green, then an inert commit and a dirtying edit. Without the dirt the tree would carry, so the row still tells the two cases apart. Correct.
- "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor": the priming `--all` now runs on the fresh pair, whose HEAD is the merge base. So the whole-green ancestor is the merge base itself. The assertion that the derivation comes from `f.mergeBase` is then met both when a dirty tree ignores the green ancestor and when it wrongly uses it. **Not correct.**
- "tiers on one base share one derivation line…": tier0's green is recorded after the `Seen = 3` commit, so it differs from the merge base. The split assertion still tells them apart. Correct.
- "a diff run diffs from the newest whole-green ancestor…": a commit lands before each gate call, and `green` differs from the merge base. Correct.
- `newScenarioHub` and `scenarioHub.clean`: the hub starts dirty, and `clean` removes the dirt file once per row. No row calls `clean` twice. Correct.
- `scenarioHub.writeModule`: it mutates the caller's map, but every caller now passes a fresh literal per row, including the moved `guarded` module. Correct.
- `scenarioHub.acquire`, `requireFree`, `writeEnvFile`, `wrapComment`: per-hub paths; cleanup is now dropped with the hub. Correct.
- `withSharedBuild`, `buildRecords`, `closingOf` at package level: bodies are unchanged, and `crashEnv` is cloned before `append` (len = cap = 1, so `append` allocates regardless). Correct.
- Refusals table, `setup`/`wantError(f)`:
  - The slot-dir row's restore runs before `requireFree`, so `Holders` reads a real directory again.
  - The machine-file row's restore removes the file.
  - `wantError` is evaluated after `restore`, but it reads only `f`'s paths.

  Correct.
- The crash and orphan nested tables build one hub per nested row, and `setup` takes `f`. Correct.
- "a real go test tier in the nested module…": it now creates `nestedModule` itself, which it used to inherit from the earlier `-C` row. Correct.

**Findings:**

1. HIGH, design, `internal/gatecli/scopes_integration_test.go:1110-1119`. The row "a dirty worktree diffs from the parent merge base although the record holds a whole-green ancestor" now passes vacuously.
   - On the fresh hub, the priming `--all` (line 1113) runs at the pair's HEAD, which equals `f.mergeBase`. So the whole-green ancestor is the merge base itself, and the derivation check at line 1118 cannot fail when a dirty tree wrongly takes the green ancestor.
   - At the base (`git show b65f0be65`, lines 1062-1070), the preceding row had left commits beyond the merge base, so the green ancestor differed from it.
   - Fix: commit a change (for example to `other/other.go`) before the `--all` run, so the green ancestor differs from `f.mergeBase`.

No other correctness findings.

Tool calls:
- One bash call: grep of the HEAD row and `git show` of the base's two derivation rows, to confirm the row's exact HEAD lines and its base starting state at line level. My context held these only as diff hunks.
