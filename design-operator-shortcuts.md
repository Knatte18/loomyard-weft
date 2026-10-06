# Short top-level commands for what the operator types most

`operator-shortcuts` · Note · feature, orch, reed

Idea (operator, 2026-10-06): top-level shortcuts for the lyx commands the operator types most, so the common path is one short word.

- `lyx attach`: the same as `lyx reed attach`.
- One command for the prime that starts the orch (`lyx orch start`, which also spawns its watcher) and then attaches the terminal to the prime's reed session (`lyx reed attach`); the orch always runs in the prime.
  Name open: `lyx spawn` was floated; `lyx up` would clash in meaning with `lyx reed up`.
  It fits the decision recorded in task `bugfix-sessions` that start verbs never attach: this command is the one place that does both, by calling `lyx reed attach` explicitly.

Shortcuts are thin wrappers: same flags, same JSON envelope, listed in `lyx --help`, and each names the full command in its help so the two never drift.
More shortcuts get added here as the operator notices them.

Related notes: `ide-prime-setup` (a VS Code task that starts the orch) and `ide-reopen-prime` (reopen the prime window with one command).
