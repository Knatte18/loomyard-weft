# Creel delivers mail to a working Claude strand via a PostToolUse hook

A Claude strand in one long turn, such as a darn writer, now receives its creel mail at its next tool boundary.
Before, mail reached it only when the turn ended and the strand went idle, and `lym creel interrupt` would have ended the turn and lost a tool call in flight, such as a gate run.

Closes #871

## What changes

- `internal/creelengine` keeps an empty flag file per strand that holds unread mail, at `<hub>/_board/.lym/creel/flags/<flag name>`.
  The flag name is the strand name with every `:` turned into `.`, formed in `internal/agentname`; the directory name is a `lymdirs` constant and the path comes from `internal/hubgeom`.
  The store recomputes the flag set at the end of `Store.do`, under its mutex, and creates or removes only the files that differ.
  The daemon reconciles the whole directory at start, so a stale or missing flag is repaired.
  Programs get no flag, and flags are derived state, never journaled.
- A hidden verb `lym creel hook-read <engine>` claims the caller strand's unread mail oldest first, agent mail included.
  It prints one `[creel] #<id> from <sender>: <body>` line per message, the same line the idle path types for a notice, as the provider's tool-boundary context.
  Messages are journaled read only after the verb wrote its output.
  Under a hold it claims only the hold owner's mail, and while a pause covers the strand it claims nothing.
- `claudeengine` renders a `PostToolUse` hook with no matcher into every hub Claude strand's settings.
  The hook is plain `sh`: it forms the flag name from `$LYM_STRAND_NAME` with parameter expansion alone, exits 0 at once when the flag is absent, and otherwise runs `lym creel hook-read claude` from `$CLAUDE_PROJECT_DIR`.
  A standalone run, told no board, gets no hook.
- Codex strands are unchanged and keep the pane path.

## Design decisions

- **Provider seam.**
  A new optional capability `shuttleengine.ToolBoundaryContextRenderer` renders text as the provider's hook output; Claude implements it as `hookSpecificOutput.additionalContext`.
  `shuttleengines` hands it out by engine name, so Claude's JSON shape stays in `claudeengine`.
- **One claim for both paths.**
  The idle pane path now reserves its notice in the store before typing it and releases the reservation when the send was busy.
  The hook and the pane path claim through the same reservation, so each message is delivered once; before, a hook read between the pane path's typing and its journaled read would have handed the notice out twice.
- **Budget.**
  One claim hands out lines up to 8 KiB, and always at least one message; the rest stays unread with its flag standing for the next boundary.
  A single message over the budget is handed over whole.
- **Failure output.**
  On a refusal or daemon error `hook-read` writes nothing to standard output, writes the error to standard error and exits 1, with a 10 second call timeout.
  Standard output of a `PostToolUse` hook is parsed as hook JSON, so the usual JSON error envelope would be misread.
- **Agent mail guard.**
  The idle path sends agent mail and long messages as a reminder line naming `lym creel read`, so an agent's text cannot pose as an instruction.
  The hook skips that guard and hands the text over; the line still opens with creel's stamped sender, and only mail the strand already holds is handed over.
  The idle path's own rules are unchanged.

## Known limits

- A flag left standing while no daemon runs makes the hook start `lym` at each tool call, which starts the daemon and reconciles the flags.
- The flag stands while a hold mutes non-owner mail, so the hook starts `lym` at each tool call of a held strand until the hold ends.

## To check by hand

Not run: a live Claude Code session with a real `PostToolUse` hook.
The change assumes Claude Code passes the session environment (`LYM_STRAND_NAME`, `PATH`) to hook processes, sets `CLAUDE_PROJECT_DIR`, runs the hook inside the strand's process tree so creel's peer-credential identity resolves it, and passes 8 KiB of additional context to the model unshortened.
Send mail to a working strand mid-turn and confirm it arrives after the next tool call, once, without ending the call.

Docs are updated in the `doc.go` of each changed package and in `docs/overview.md`.
