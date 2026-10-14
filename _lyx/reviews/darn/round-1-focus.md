---
round: 1
focus:
  - "Scope against the board entry: the summary moves the rows from right after `Worktree-Create` to after `Seed-Child`. Settle whether the amended entry or a decision-record `Added after Discussion` entry licenses that placement, or whether it is an unrecorded parent correction (class `decision`)."
  - "Removed code: check that nothing still references `InnerRunDeps.OpenTerminal`, `battenshed.OpenTerminalOnce` or the terminal-opened once-marker (docs, help text, tests, other recipes), and that the removal is licensed by the entry."
  - "`ideengine.SeedTask` and `ideengine.OpenTask`: settle whether they share the config write with `lyx ide spawn` without changing its behavior, and whether `SeedTask` really leaves the user's key bindings alone."
  - "`battenCLI.switchedActions` and `--window`: settle whether `--open-terminal` forces `Open-Window` on for that run only, and whether a flag-read failure is a row error while an action failure only warns."
  - "Docs and PATTERN: settle whether the docs the repository's CLAUDE.md names for a new config module and changed CLI behavior land in this diff (`doc.go`, `docs/overview.md`, `PATTERN-config-strictness`), and whether `cmd/lyx/testdata/test-budget.yaml` needs an edit for the new top-level tests."
  - "Test economy: settle whether the new tests in `switched_test.go`, `entries_switched_test.go`, `wire_seams_test.go` and the integration tests duplicate each other or the conformance table, and whether any added behavior (the resumed-past-`Open-Window` case, the unreadable-flag error) has no coverage."
  - "Summary claims: check each claim in `_lyx/landing/summary.md` (defaults `open_window: true`, `seed_ide: true`, `open_ide: false`; local-layer openness; `LoadOrTemplate` fallback) against the diff."
---

Review range comes from `product.parent` in `_lyx/shed/seed-vscode-config/status.json`, which is `main`.
