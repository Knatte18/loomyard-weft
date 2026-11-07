---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: internal/agentconfig/config_test.go:94
    summary: The UnknownEffort row asserts only that Load wraps ErrUnusable, so the claim that the refusal names the key has no test
    fix: In TestLoad_RefusesUnusableContent add an optional wantKey field to the table, set it to efforts.haiku on the UnknownEffort row, and in the loop fail unless err.Error() contains it when it is non-empty.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/command.go:402
    summary: The OpenLaunch doc comment says it validates the effort but never states that a spec with no effort takes the agents.yaml entry for its model alias
    fix: In the OpenLaunch doc comment, before the sentence that begins It validates the effort, add one sentence that a spec naming no effort takes the agents.yaml effort for its model, a full Claude model id resolving to its alias, else the default entry.
---

### [LOW] The refusal test does not pin that the key is named

**Location:** internal/agentconfig/config_test.go:94
**Issue:** `checkEfforts` returns an error built by `unusable(effortsKey+"."+key, ...)`, so the key is named in the code.
The focus asks that an effort outside the vocabulary be refused at load naming the key.
The `UnknownEffort` row passes as long as any content failure wraps `ErrUnusable` and `configengine.ErrInvalid`.
A change that dropped the key from the message would pass every row.

### [LOW] OpenLaunch's doc comment omits the effort default it now applies

**Location:** internal/shuttleengine/claudeengine/command.go:402
**Issue:** `OpenLaunch` now sets `spec.Effort` through `effortFor(cfg.Agents, "", spec.Effort, spec.Model)`, so a spec with no effort gets the agents.yaml default or its model alias entry.
The doc comment lists what the function validates and writes but not this behavior, which decides whether the function fits a caller that wants an unpinned effort.

## Settled points

- **Default effort and model alias on an opened session.** `OpenLaunch` calls `effortFor` with an empty role.
  `effortFor` returns an explicit effort unchanged, else `agents.Effort("", modelAlias(model))`.
  `modelAlias` cuts the `claude-` prefix and the first dash segment, so `claude-haiku-5-5` gives `haiku` and `claude-fable-5-1` gives `fable`.
  `TestOpenLaunch` rows "a spec with no effort takes the agents.yaml default" and "a full model id takes the effort of its alias" check `--effort` on both the fresh and the resume line against the template.
  The first row expects `medium` and fails if the default lookup breaks.
  The second expects `high`, where the default would give `medium`, and fails if the alias resolution breaks.
  The alias row uses a fable id, not a haiku id; that is the same code path.
- **Effort vocabulary.** `agentconfig.effortVocabulary` is the one declaration.
  `Efforts()` returns a clone, and `validateEffort` in claudeengine reads it, so the old `validEfforts` map is gone.
  `checkEfforts` refuses an effort outside it at load with the key named, and the `UnknownEffort` row covers that refusal (see F1 for the key).
- **No check left unreachable.** The subagent effort check in `subagentEffortEnv` was dropped in 832318b9, since `Load` now refuses a bad effort first.
  `validateEffort` in `Prepare` and `OpenLaunch` stays reachable, because a model-spec can still name an effort that never passes through agents.yaml.
- **Config help.** The net diff removes the pre-existing help line `A value the loader refuses is refused, naming the key.` from `buildConfigLong`.
  The commit message of 0cabddd0 states the reason: the `Long` cap of `maxLongBytes` (2000) in `cmd/lym/clitree_test.go` forced the deletion, because the hub-wide and known-module lists grow with the new `agents` module.
  The goal does not touch config help, and the stated reason is the one the focus asks for, so this is not a defect.
  The test change in `configcli_test.go` selects the `batcher` menu index through `configreg.Names()`, which keeps the menu test independent of the module order the new module changes.
- **Declared PATTERN changes.** `pattern/PATTERN-config-layers.md` lists `agentconfig` among the degrading callers, and `pattern/PATTERN-leaf-packages.md` adds the `internal/agentconfig` import set.
  The two diffs match how `Load` calls `configengine.LoadOrTemplate` and what the package imports.
