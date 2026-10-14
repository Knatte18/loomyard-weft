# Support log — seed-vscode-config

## Interview

A darn run holds no interview; the board entry is the whole statement of the change.

## Rejected alternatives

- **Rows directly after `Worktree-Create`.**
  Rejected: the task's `run_settings` reach the pair's local layer only in `Seed-Child`, so the flags could not be set per task on a first run.
- **Overlaying the board task's `run_settings` onto the flag read inside the row.**
  Rejected: it stacks a config layer outside `configengine`, against `PATTERN-config-layers`, and duplicates `Seed-Child`'s local-layer write.
- **Three producer types, one per row.**
  Rejected: identical Calls three times; one type under three engine names gives the same independent rows.
- **Halting the run when an action fails.**
  Rejected: a missing window or VS Code is no reason to stop a task; today's window open only warns.
- **Reading the flags once at wire time from the prime's config.**
  Rejected: the per-task local layer lives in the pair and is written after wiring.
- **A new package for the batten config module.**
  Rejected: `battenshed` already owns batten's producers; widening its allowlist by `configengine` is smaller than a package of its own.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Where do the rows sit, given `run_settings` land in `Seed-Child`? Self-pick: after `Seed-Child`, before `Run-Shed`.
- Q: Does a failed action halt the run? Self-pick: no, it warns and the row is Done.
- Q: Does `open_ide` also seed the user key bindings? Self-pick: no, launch only; the key bindings stay with `lyx ide`.
- Q: Is `batten.yaml` hub-wide? Self-pick: per-worktree, open to the local layer.
- Q: How does `--window` override `open_window`? Self-pick: the hidden `--open-terminal` it passes forces `open_window` true for that run.
