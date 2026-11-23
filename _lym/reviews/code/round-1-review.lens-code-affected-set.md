```yaml
lens: code-affected-set
outcome: reported
```

code-affected-set lens: checked everything outside the diff's lines that reads or describes the verify-budget notice text, which now opens with `loom <slug>: `.

Per change, the set found and whether it is covered:

- **Notice text sent through `treeScopeSettings`' `Notify` (`internal/loomcli/darnwiring.go:270-275`)**
  - Production caller `wireTreeScope` (`darnwiring.go:216-248`), reached from `wiring.go:1339` for every recipe whose verify source is tree_scope: covered. It passes the budget through unchanged.
  - Producer `budgetClock.noticeIfSpent` and `verifyBudgetNotice` (`internal/loomshed/verifybudget.go`): covered. They are unchanged, and the prefix wraps their line from outside. The doc at `internal/loomshed/doc.go:62` only says a notice goes to the parent, so it is still true.
  - Hardener branch (`darnwiring.go:258-261`): unaffected, since it sets no budget.
  - Sibling notices with the same form: `raddlewiring.go:244-248` and `wiring.go:993-997` (`seedSlug(location.WorktreeName)`, which returns the name unchanged, `seedinput.go:77-79`). The new line matches them, so covered.
  - Mail run tag `NotifyParent` (`internal/orchcli/notify.go:50-53`) already uses `pair.WorktreeName`. The prefix uses the same value, so they agree: covered.
  - Tests: `TestVerifyBudgetNotice_NamesTheRun` (`notifystamp_integration_test.go:150-178`) asserts the new subject. No other test asserts the notice text. `wiring_darn_test.go:91` only seeds `verify_budget_min`. Covered.
  - Docs and comments describing the notice: `contracts/recipes/darn-recipe.yaml:118-119`, `internal/darnengine/template.yaml:4`, `internal/shedrecipe/recipe.go:122`, `darnwiring.go:212` and `darnwiring.go:251-256`. None states the line's wording, so none is made stale. The doc comment the diff adds at `darnwiring.go:254` states the new behaviour. Covered.
  - Stencils: none describes this notice's text. A search of the tree for "verify gate has run past" and "loom <slug>" found nothing in `contracts/`, so the "Stencil wording" check has nothing to apply to.
- **Test file header (`notifystamp_integration_test.go:1-2`)**: updated to name both stamps the file proves. Covered.
- **The repository's own rules**
  - `PATTERN-test-speed`: the new test opens with `tiergate.Require(t, 1)`, and its doc comment says why it skips `t.Parallel` (process-global `t.Setenv`). Covered.
  - `PATTERN-agent-addressing` and `PATTERN-shuttle-provider-seam`: the notice still goes through `orchcli` to creel. Covered.
- **Declared PATTERN changes**: the directive declares none, so there is nothing to check.
- **Commit messages and departures**: the one commit, ef4006a7f, carries a `Root-Cause:` trailer and no `Pattern-Departure:` or `Loosened-Check:` trailer. No existing test assertion was widened. Nothing to judge here; whether the root cause is right belongs to the other lenses.

Findings: none.

Tool calls:
1. A tree-wide grep for verify-budget, `verify_budget` and "run past its" mentions in md, go, yaml and tmpl files, to find the docs, stencils, tests and fixtures describing the notice. The manifest held no such search.
2. Read `contracts/recipes/darn-recipe.yaml:114-122` and `internal/loomcli/wiring_darn_test.go:80-110`, and grepped `contracts`, `docs`, `pattern` and `PATTERN.md` for `loom <slug>`. This checked whether the hits from call 1 state the notice's wording; the manifest did not hold these files.
