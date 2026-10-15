---
verdict: APPROVED
---

Range: `git diff $(git merge-base main HEAD)..HEAD`, with `product.parent` = `main` from `_lyx/shed/seed-ide-interactive-chain/status.json`.

The diff makes the change the board entry and its operator decision ask for.
The folderOpen chain is gone from the one shared VS Code seeding path.

## Focus questions settled

- **Leftover references.**
  A repo-wide grep for `TaskChain`, `claudePath`, `folderOpen`, `runOn`, `writeInteractive*`, `Start Claude`, "interactive chain" and "launch chain" finds no remaining reference in code, `doc.go`, `docs/overview.md` or PATTERN.
  The remaining `--if-absent` and `--unless-name` hits are reed's own flags and tests, which the record scopes out.
- **The single task.**
  `internal/vscode/config.go` writes one `reed attach` task with the stamped absolute `lyx` path (bare `lyx` when empty), and no `runOptions` or `dependsOn`.
  tasks.json is still overwritten on every write.
  The tracked-tasks.json skip and the info/exclude handling live in `ideengine.writeVSCodeConfig` and are unchanged apart from the dropped parameters.
- **Unit-test coverage.**
  The `internal/vscode/config_test.go` rewrite keeps the settings.json keep, the tasks.json overwrite, the bare-name fallback and the untouched `.gitignore`.
  The dropped rows only pinned the deleted chains.
  The tracked-tasks.json skip and info/exclude coverage stay in the ideengine integration tests, whose `assertAttachTaskOnly` checks the shape through the shared path rather than repeating the unit test's presentation and args assertions.
- **SeedTask coverage.**
  `checkTaskSlugOpensBareFolder` now calls `assertAttachTaskOnly` after `SeedTask`, so the Seed-IDE path is exercised for real.
- **Change description.**
  Every claim in `_lyx/landing/summary.md`, including the Tests section, holds against the diff.

No finding.
