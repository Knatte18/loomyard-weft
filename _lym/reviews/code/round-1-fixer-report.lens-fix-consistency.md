Task: check the fix-consistency lens, meaning whether any sibling text or twin was left stale by the change set.

**LOW** — `internal/shuttlecli/hookpython.go:33-39`, the verb's `Long` help text.
- It still reads "In a hub that is the hub's layers; in a standalone run, which has no hub, it is the state directory".
- The fix changed `shuttleConfigDirs` (F4) so a hub whose Bolt checkout lacks `_lym` also reads the state directory, and updated that function's comment, `summary.md` and the tests.
- The help text was left as before. It says nothing wrong about a healthy hub, but it omits the new rule, so a reader of `--help` would not learn it.
- The sibling comment above `shuttleConfigDirs` says "a hub whose Bolt checkout holds `_lym`".

No other stale sibling found. I grepped the repo for `hook-python-allowed`, "pure function of the row", "holds no path" and the other bare-`lym` wording.
- The `claudeengine` and `codexengine` `doc.go` files, `docs/overview.md` (both paragraphs), `shuttlecli/cli.go` and `summary.md` all agree with the lym-by-path change.
- The Claude and Codex twins (`pythonAllowedCheck`, `guardCommand`/`renderShellGuard`, `guardCandidates`, `toolHooks`/`launchArtifacts`, TestMain stand-ins) were changed in matching ways.

Calls made:
1. One Bash call combining two greps: the first for `hook-python-allowed` and bare-`lym` wording in non-test `.go`, `.md` and `.yaml` files, to find stale siblings outside the changed files; the second for the "pure function of the row" and "on PATH" wording. My context held only the files the fixer edited.
