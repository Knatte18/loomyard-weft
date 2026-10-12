# Paused runs in the overview, hold-notice end, chunked paste, operator shortcuts

`operator-followups` · Task · enhancement, feature, reed, shuttle, batten, orch

Depends on: [`lyx-halt`](design-lyx-halt.md), [`runs-overview`](design-runs-overview.md)

Four small follow-ups, bundled as one task (operator, 2026-10-12).

## Runs overview shows a paused run as paused (was `runs-overview-paused`)

### Seen (2026-10-12, from `lyx-halt`'s parent review)

`lyx-halt` adds a pause, hub-wide or per run, recorded with a `paused_since` time.
The runs overview pane from `runs-overview` reads run states without it, so a paused run shows as running.

### Want

The runs overview shows a paused run as paused, with the time since it was paused, reading `lyx-halt`'s record.

## End the hold-notice deferral when the advisor is held or dead (was `hold-deferral-end-on-held`)

Seen in `shuttle-startup-ready` (PR #557, 2026-10-11): Part 3 shortened `holdNoticeMaxDeferral` from 20 to 10 minutes (`hold.go`).
The alternative from the same idea was not built: end the deferral as soon as the advisor the chair waits on is itself held or dead, instead of waiting out the cap.

## A paste over tmux's message size fails whole (was `paste-size-limit`)

Seen at `pane-inject-safety` PR-Gate (PR #548): the one-command set-buffer + paste-buffer + Enter path sends the text in one tmux command, so a text over tmux's command size limit fails the whole send.

Idea: load the buffer in chunks (or from a file with `load-buffer`) before the single paste-and-Enter step, and test a text above the limit.

## Operator shortcuts built into lyx (was `operator-shortcuts`)

Idea (operator, 2026-10-06): top-level shortcuts for the lyx commands the operator types most, declared in one simple file in this repo rather than as Go commands.
The operator now watches runs in Konsole windows that `lyx batten run --window` opens, not in a VS Code terminal (2026-10-12); the shortcuts are still wanted.

Examples:

- `attach` expands to `reed attach`.
- One shortcut for the prime runs `orch start`, then `reed attach`; the orch always runs in the prime.
  Name open: `spawn` was floated; `up` would read like `reed up`.
  It fits the decision recorded in task `bugfix-sessions` that start verbs never attach: the shortcut is where both happen, by naming `reed attach` explicitly.

### Shape (operator, 2026-10-06)

- The shortcuts are part of the deployed `lyx` binary, the same on every hub and repo: a committed file in the loomyard repo, embedded at build time (`go:embed`), never runtime config.
  No hub, repo or user can add or change one; changing a shortcut is a commit here and a deploy.
- The file maps a shortcut name to one lyx argv, or to a list of argvs run in order that stops at the first failure.
  Only lyx commands, never a shell string.
- At root construction lyx adds one cobra command per shortcut, in its own help group, whose `Short` names the expansion; running it re-dispatches the expanded args through the same root, so flags, envelopes and exit codes are the real command's.
  Extra args after the shortcut append to the last argv.
- A test fails when a shortcut names an existing command or another shortcut, or expands to a command that does not exist, so a broken file never builds into a binary that passes CI.

### Open

- Whether a shortcut may carry flags of its own or only append args.

Related notes: `ide-prime-setup` (a VS Code task that starts the orch) and `ide-reopen-prime` (reopen the prime window with one command).
