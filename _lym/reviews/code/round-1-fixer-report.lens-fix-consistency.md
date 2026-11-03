```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-consistency lens, meaning whether the fixer's change set still agrees with its surroundings.

No `NewPreflightWithBatcher` reference remains anywhere, so the F7 header fix left no stale twin under that name.

**LOW: `internal/loomcli/cli.go:41` comment**
- Changed text it concerns: F7, the file header at `internal/preflightshed/preflight.go:1` now naming `NewPreflightWithChecks`.
- Summary: a twin of the corrected reference was left as it was.
- Evidence: the comment says `preflightEntry` "makes the same preflightshed.NewPreflight(name, env.Cwd) call". `internal/shedrecipe/entries_simple.go:25` now calls `NewPreflightWithChecks(row.Name, env.Cwd, Checks{...})`.
- That comment predates this round and `NewPreflight` still exists, so it is a stale cross-reference rather than a regression from the fix.

**LOW: reason prefix on a table-build failure (F3)**
- Changed text: the wrap in the `PreflightStencils` closure at `internal/loomcli/darnwiring.go:264`.
- Summary: the Stuck reason for a table-build failure still opens with "seat stencils cannot be filled: ".
- Evidence: `stencilsFailure` in `internal/preflightshed/preflight.go` always prepends `stencilsReasonPrefix`. A table-build failure therefore reads "seat stencils cannot be filled: build the darn writer table: …; way forward: …".
- The review's stated goal was that only a `CheckMarkers` failure reads as an unfillable stencil. That holds only inside the error text, not in the reason prefix.
- The wrap now names the fault and carries a way forward, so this is a residual, not a regression.
- The `PreflightStencils` doc in `internal/shedrecipe/recipe.go:85` and the `Checks.Stencils` doc ("refuses a stencil marker no value fills") describe only the marker refusal. They are slightly narrower than what the check now returns.

Everything else agrees with its siblings:
- The `CheckMarkers` doc and the `fills` doc agree on non-blank values.
- `Validate` already refuses a blank value that is not Optional, so the `fills` change adds no contradiction.
- The `TestTable_CheckMarkers` rows follow the file's existing error-text format, and its header still describes the file.
- The `wiring_darn_test.go` doc comments and the `export_test`/envkit seam lists are untouched by the new subtests.

Tool calls made:
1. `grep -rn "NewPreflightWithBatcher"` over `.go` and `.md` files: checked for stale twins of the F7 header fix, which my context lacked.
2. `sed -n 38,46p internal/loomcli/cli.go`: read the `NewPreflight` mention, which my context lacked.
