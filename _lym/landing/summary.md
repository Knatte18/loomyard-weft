# Creel delivers mail to a working Claude strand at its next tool boundary

A Claude strand in a long turn, such as a darn writer, used to see creel mail only once its turn ended, and `lym creel interrupt` would end the turn and lose a tool call in flight.
Creel now hands the mail over at the strand's next tool boundary through a `PostToolUse` hook, as SendMessage does, without interrupting the running call (issue #871).
The idle pane path stays for a strand with no tool call coming.

## What changes

- **Mail flags.**
  While a strand holds unread mail, creel keeps an empty flag file for it in `<hub>/_board/.lym/creel/flags/`, named by the new `agentname.FileName` (the full name with `:` turned into `.`, valid on Windows).
  `Store.do` resyncs the flags whenever its function journaled a record, so a flag goes when the last unread message is read, forwarded, withdrawn or returned.
  The daemon reconciles the directory to the pool at start (`Store.ReconcileFlags`).
  Programs get no flag, and flags are never journaled.
- **The claim.**
  `Store.ClaimAtBoundary`, served by the new `claim` protocol verb, hands a strand its unread, unreserved mail oldest first, agent mail included, through the read's reserve-then-acknowledge step.
  Under a hold it hands out only the owner's mail, under a pause nothing, and it stops at an 8 KiB budget of lines while always handing out at least one message.
  The daemon's read and claim share one hand-out function, `handOut`.
- **One delivery per message.**
  The pane path now reserves a notice before it types it (`reserveNotice`), releases it after a busy send, and journals it read after the send.
  Before this change a notice being typed was unreserved, so a concurrent claim could have handed it out a second time.
- **The verb.**
  Hidden `lym creel hook-read <engine>` claims the caller's mail and prints it as that engine's tool-boundary context, one `MessageLine` (formerly `composeNotice`) per message.
  It prints nothing when nothing was claimed, acknowledges only after writing, and sends refusals to stderr with exit 1 and an empty stdout, because the provider parses the hook's stdout as JSON.
- **The hook.**
  A new optional capability `shuttleengine.ToolBoundaryContextRenderer` renders the context; Claude implements it as `PostToolUse` `hookSpecificOutput.additionalContext`, and `shuttleengines.ToolBoundaryRenderer` hands it out by engine name.
  `claudeengine` installs a matcher-less `PostToolUse` hook whenever its engine is told a hub board directory.
  The hook is plain `sh` that forms the flag name from `$LYM_STRAND_NAME` by parameter expansion alone and exits at once when the flag is absent.
  Only when the flag stands does it change to `$CLAUDE_PROJECT_DIR` and run `lym creel hook-read claude`.
  `claudeengine.CreelFlagDir` derives the directory from the board directory, and `TestCreelGeometry` pins it to `hubgeom.CreelGeometry`'s `FlagDir`.

## Design decisions

- The hook tests a per-strand file rather than a hub-wide list, so the no-mail path forks nothing even while other strands, the orch included, hold mail.
- The flag stands while a hold mutes non-owner mail, as the board entry defines it (unread mail), so a held strand starts `lym` at each tool call until the hold ends; holds last at most an hour.
- The budget keeps one tool result's added context bounded; a single message over it is still handed over whole.
- The hook path carries agent mail in full, where the pane path only reminds it, as the board entry asks; each line opens with the sender creel stamps.
- Codex strands get no hook and keep the pane path, as scoped.

## Tests

- `creelengine`: flag set, clear, forward, withdraw, program and reconcile rows; claim order, acknowledgement, release, budget, hold, pause and refusal rows; a notice being typed is not claimed; the claim verb over a real socket with its flag following the mail.
- `claudeengine`: the hook command in the settings, absent for a standalone run; the `PostToolUse` context JSON; the rendered hook run under `sh` with a stub `lym` for a standing flag, another strand's flag, an empty name and an unset name.
- `creelcli`: `hook-read claude` output, nothing on a second call, and an unknown engine refused on stderr.
- `shuttleengines`, `hubgeom` and `agentname`: the renderer lookup, the flag directory pin and `FileName`.
