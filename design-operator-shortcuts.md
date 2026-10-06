# Shortcut commands built into lyx from one committed file

`operator-shortcuts` · Note · feature, orch, reed

Idea (operator, 2026-10-06): top-level shortcuts for the lyx commands the operator types most, declared in one simple file in this repo rather than as Go commands.

Examples:

- `attach` expands to `reed attach`.
- One shortcut for the prime runs `orch start`, then `reed attach`; the orch always runs in the prime.
  Name open: `spawn` was floated; `up` would read like `reed up`.
  It fits the decision recorded in task `bugfix-sessions` that start verbs never attach: the shortcut is where both happen, by naming `reed attach` explicitly.

## Shape (operator, 2026-10-06)

- The shortcuts are part of the deployed `lyx` binary, the same on every hub and repo: a committed file in the loomyard repo, embedded at build time (`go:embed`), never runtime config.
  No hub, repo or user can add or change one; changing a shortcut is a commit here and a deploy.
- The file maps a shortcut name to one lyx argv, or to a list of argvs run in order that stops at the first failure.
  Only lyx commands, never a shell string.
- At root construction lyx adds one cobra command per shortcut, in its own help group, whose `Short` names the expansion; running it re-dispatches the expanded args through the same root, so flags, envelopes and exit codes are the real command's.
  Extra args after the shortcut append to the last argv.
- A test fails when a shortcut names an existing command or another shortcut, or expands to a command that does not exist, so a broken file never builds into a binary that passes CI.

## Open

- Whether a shortcut may carry flags of its own or only append args.

More shortcuts get added here as the operator notices them.
Related notes: `ide-prime-setup` (a VS Code task that starts the orch) and `ide-reopen-prime` (reopen the prime window with one command).
