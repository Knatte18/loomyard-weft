```yaml
lens: fix-code-correctness
outcome: reported
```

No findings.

- **New row `FullAccessSandboxMode`:**
  - `setup` writes `sandbox_mode: danger-full-access` to the worktree config. `OpenLaunch` loads it through `loadConfigOrRefuse` and passes `codex.SandboxMode` into `launchLine` (`codexengine.go:186`).
  - The want side builds `codexCommand` with `sandboxFullAccess`. That emits `-s danger-full-access` and skips the network and writable-roots flags (`command.go:86-92`).
  - If `OpenLaunch` hardcoded `workspace-write`, the command strings would differ and `reflect.DeepEqual` would fail, so the row catches the defect.
- **Default rows:** An empty `wantSandboxMode` falls back to `sandboxWorkspaceWrite`, so the other rows keep their old expectation.
- **Failure path of the writes:** `writeFile` does not create parent directories (`fixture_test.go:164`). The setup uses the same call as `prepare_test.go:111`, and the fixer's `lym gate test ./internal/shuttleengine/codexengine` run passed, so the directory exists in the fixture.
- **Imports and callers:** The `configengine` import is used by the new row. No signature changed, so there are no callers to re-check.
- **Doc edits:** The `PATTERN-lym-files.md` and `docs/overview.md` edits are wording only. They match the review's `fix` text.

Further tool calls (2):
- `grep -A8 "^func writeFile"` in the package tests, and `sed -n 80,100p command.go`. My context lacked the `writeFile` body and `codexCommand`'s branch on `sandboxMode`.
