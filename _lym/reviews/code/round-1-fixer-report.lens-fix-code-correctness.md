```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the fixer's changed code through the fix-code-correctness lens (`shuttleConfigDirs`, the verb's error output, the new tests).

I walked each branch of `shuttleConfigDirs` with one concrete input.
- **Hub worktree with a Bolt checkout:** returns `hubDirs`, unchanged from before.
- **Non-git standalone target whose state dir exists:** `ResolveWithin` fails, `target` is `cwd`, `Derive` and `Stat` succeed, so it loads `Dirs{Anchor: stateDir}`.
- **Non-git directory with no state dir:** returns the zero `Dirs` and `resolveErr`, so the verb prints a JSON error and exits 1.
- **Git-repo standalone target:** `ResolveWithin` succeeds and `Board` is empty. `target` becomes `WorktreePath()` (the repo root, which is also where a launch sets its pane cwd). A state dir that exists is used, and otherwise it falls back to `ConfigDirs(layout)` as before.
- **`Derive` or `Stat` fails:** falls through to the previous behaviour or the resolution error. Nothing is left unhandled.

The caller (`pythonAllowed`) and the verb's `RunE` still hold: `allowed` is false on every error path, so the verb exits 1. Both tiers ran green.

Tests fail with the defect in place:
- **Standalone test:** `PythonAllowed` expects 0. Without the fallback a non-git temp dir errors and exits 1, so it fails.
- **`AllowedFromBelowTheAnchor`:** needs `ResolveWithin`, so it fails under a strict cwd gate.
- **Hub rows and `OutsideAWorktree`:** the rows assert empty output, and the error row asserts the JSON object, so they fail if the verb reverts to plain text.
- **Orch test:** the `DenyPythonFalse` row fails if the wiring loads the template instead of the hub's default layer. The `DenyPythonTrue` row guards against a vacuous pass.
- **Parallel and env interaction:** `t.Setenv` in the serial standalone test cannot overlap the parallel hub test, because Go resumes parallel tests only after the serial ones finish.

Findings:

1. **LOW**, `internal/orchcli/denypython_integration_test.go`, the `pythonNotice` loop. It keeps only the last row hanging on `SwitchDenyPython`, but the engine comments speak of "python rows" in the plural. If more than one row hangs on the switch, the `DenyPythonFalse` row checks only that last notice. A leftover notice on another row would pass.
   - Evidence: `for _, row := range shellguard.Rows("", nil) { if row.Switch == shellguard.SwitchDenyPython { pythonNotice = row.NoticeFor(engine) } }` overwrites on each match.
   - A fix would collect every matching notice and assert on each.

2. **LOW**, `internal/configcli/configcli.go`, the edited help line. "…drops the notice\nfrom sessions launched afterwards, from the prime:" reads as if "from the prime" belongs to the notice drop. It is meant to introduce the example command. This is wording only, not behaviour.

Further tool calls: none, because the context already held every changed file and the gate output.
