# claudeengine: 5m cache TTL on open, and no slash command behind a space

Two fixes in `internal/shuttleengine`, `claudeengine` and the verified-send path.

## Open sessions use the role's cache TTL

`buildOpenCmd` set no `CLAUDE_CODE_PROMPT_CACHE_TTL`, so a session from `lym shuttle open` wrote its prompt cache at Claude Code's 1h default.
`Claude.OpenLaunch` now resolves the TTL for `OpenSpec.Role` from `claude.yaml`, as a run does.
An invalid TTL anywhere in `claude.yaml` refuses the open before any file is written.
The fresh line and the resume line both start with a statement exporting the variable.
The resume line falls back to the fresh line, so the export sits in front of the joined pair.
It uses `sh.ExportEnv` and `sh.Chain`, which keeps both shell dialects covered.
Every window launched through `OpenLaunch` gets the TTL, not only `lym shuttle open`.

## A slash command is not pasted behind whitespace

`isBlankBoxInterior` counts whitespace as an empty box, so a stray space passed the idle check and sat in front of the pasted `/compact`, `/clear` or `/reload-plugins`.
Claude Code took the result as an ordinary prompt and the command silently did not run.
The source of the stray space was not found; the fix does not depend on it.

- A send that starts no turn now clears the box right before its paste, using the engine's `InputBoxClearer`.
  The clear goes only into a box that reads blank or unreadable, with no operator active in the pane.
  Text typed since the idle reading holds the send with `ErrSessionBusy`, and nothing is cleared or pasted.
- A turn start after the paste, with an empty instruction or the command's text as the instruction, fails the send with the new `ErrCommandTakenAsPrompt`.
  A turn start with another instruction, such as an operator's prompt, is not taken as a signal.
- `creelcli` passes `ErrCommandTakenAsPrompt` through as a plain send error, so creel requeues the inject and types it again once the session is idle.

Docs and the `claude.yaml` template comments are updated to match.

## To check by hand

- `clearBoxBeforeCommand` clears before the command with no activity check once the box reads blank.
  The activity query runs, but a turn started between the idle reading and the clear is not excluded.
  Confirm that is acceptable (`internal/shuttleengine/send.go`).
- `confirmPasted` fails a command send on any turn start with an empty or matching instruction, whatever turn that is.
  A turn that begins for another reason with no recorded instruction would fail a command that did run, and creel would retype it.
  Confirm that cannot happen in practice (`internal/shuttleengine/send.go`).
- Run `lym shuttle open` and check the new session's transcript shows `ephemeral_5m_input_tokens`, not `ephemeral_1h_input_tokens`.
  A real compact on a session with a space in the box was not run either.
