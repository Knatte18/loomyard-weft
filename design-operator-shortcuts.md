# Config-defined shortcut commands for what the operator types most

`operator-shortcuts` · Note · feature, orch, reed

Idea (operator, 2026-10-06): top-level shortcuts for the lyx commands the operator types most, defined in config rather than code, the way `git` and `gh` aliases work.

Examples:

- `attach` expands to `reed attach`.
- One shortcut for the prime runs `orch start`, then `reed attach`; the orch always runs in the prime.
  Name open: `spawn` was floated; `up` would read like `reed up`.
  It fits the decision recorded in task `bugfix-sessions` that start verbs never attach: the shortcut is where both happen, by naming `reed attach` explicitly.

## Shape

- A config section maps an alias name to one lyx argv, or to a list of argvs run in order that stops at the first failure.
  Only lyx commands, never a shell string, so an alias cannot hide arbitrary shell.
- At root construction lyx adds one cobra command per alias, in its own help group, whose `Short` names the expansion; running it re-dispatches the expanded args through the same root, so flags, envelopes and exit codes are the real command's.
  Extra args after the alias append to the last argv.
- An alias that names an existing command or another alias is refused at load, with the config path in the error.
- Where the config lives is open: hub-wide under `_lyx/config` (registered in `configreg`) or per operator under the user's config dir; `lyx config` edits it either way.

## Open

- The CLI-tree tests walk `newRoot()`; aliases load from config, so tests build the root without them or with a fixed fixture set.
- Whether an alias may carry flags of its own (`attach --x`) or only append args.

More shortcuts get added here as the operator notices them.
Related notes: `ide-prime-setup` (a VS Code task that starts the orch) and `ide-reopen-prime` (reopen the prime window with one command).
