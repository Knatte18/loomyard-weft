```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1, F2/F3 and F5, and check the fix covers every member.

Findings left unchecked: F4 and F6. Neither has a set premise.

**F1 (engine-name literals):** the fix covers the named site.
- `wiring.go:675` now uses `driftBaselineEngine`. No bare `"DriftBaseline"` literal is left in non-test `loomcli` code.
- LOW: two bare engine names remain beside it, and neither was touched by this fix.
  - `internal/loomcli/decision.go:1036` uses `HasEngine(recipe, "PlanWrite")`, although `planWriteEngine` exists in the `arm.go` constant block.
  - `internal/loomcli/wiring.go:661` uses `HasEngine(c.recipe, "Implementer")`.
  - They have the same defect as F1, and `decision.go:1036` has a ready constant. The review did not name them, so this is only a note.

**F2/F3 (`skipFileArgument` consumers):** the fix covers every consumer.
- The regex is used only in `withoutSkipFileArguments` and the new `argumentRemainder`, both in `diffmode.go`.
- `dropSkipFileArguments` (`diffmode.go:71`) is the only caller of `withoutSkipFileArguments`, so there is no other consumer to cover.

**F5 (guard keys):** the fix covers all four keys.
- `config.go:136-150` marks four keys as `CategoryGuard`: `diff`, `key_patterns`, `volatile_files` and `volatile_keys`.
- `runsettings_test.go` now has a refusal row for each of the four.

Tool calls:
1. One `grep` for `HasEngine(`/`recipeHasEngine(` calls with a string literal, to rerun the F1 search; my context had only the `DriftBaseline` grep.
2. One `grep` for `skipFileArgument`/`withoutSkipFileArguments`, to list the F2/F3 consumers.
3. One `sed -n 136,150p internal/driftengine/config.go`, to list the guard keys for F5; my context had only the `HubOnlyWayForward` map.

I ran calls 1 to 3 in one `Bash` invocation.
