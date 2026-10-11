# Card 1 — reed paste-submit transport op

**Create:**
- `plan:internal/reedengine#Engine.PasteSubmit` -> `func (e *Engine) PasteSubmit(guid, text string, submit bool) error`
- `plan:internal/reedengine#Engine.pasteSubmitLocked` -> `func (e *Engine) pasteSubmitLocked(paneID, text string, submit bool) error`
- `plan:internal/reedengine#pasteSubmitArgv` -> `func pasteSubmitArgv(paneID, buffer, text string, submit bool) []string`
- `plan:internal/reedengine#injectBufferName` -> `func injectBufferName() (string, error)`
- `internal/reedengine/paste_integration_test.go`

**Edit:**
- `internal/reedengine/io_test.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#Engine.resolvePaneInThisSessionLocked`
- `internal/reedengine#TmuxCmd.run`

**Intent:**
Add reed's atomic paste-submit (decision D2), the one transport op every text write into an agent pane will go through.

`Engine.PasteSubmit(guid, text, submit)` takes the op lock, requires the session, loads state and resolves the pane through `resolvePaneInThisSessionLocked` exactly as `SendText` does, then calls `pasteSubmitLocked`.
`pasteSubmitLocked` names a fresh buffer with `injectBufferName` (`lyx-inject-` plus 8 lowercase hex characters from `crypto/rand`) and runs ONE `e.tmux.run` invocation whose argv is `pasteSubmitArgv(paneID, buffer, text, submit)`.
`pasteSubmitLocked` exists so the name repair (card 3), which already holds the op lock through `withTryOpLock`, pastes without re-taking it.

`pasteSubmitArgv` is pure and returns the command list with literal one-character `";"` separator elements (the convention `chainedAttachArgv` documents, never `"\\;"`):
`set-buffer -b <buffer> -- <text>`, `;`, `paste-buffer -p -d -b <buffer> -t <paneID>`, and, only when `submit` is true, `;`, `send-keys -t <paneID> Enter`.
tmux reads an argv element ending in `;` as a command separator, so a `text` ending in `;` is passed with that final `;` written as `\;`, which tmux turns back into a literal `;`; the `--` keeps a dash-leading text from being read as a flag.
`-d` deletes the named buffer after the paste, so the operator's automatic paste-buffer stack is never touched, and `-p` makes tmux send bracketed paste when the pane's application enabled it.
The variant without the trailing Enter (`submit` false) is the D5 fallback a provider can choose.

Bound: tmux carries a command list to its server in one message, so a text longer than tmux's message limit fails the whole invocation and pastes nothing; every text lyx injects is one line (a prompt pointer, a notice, a slash command), far below it.
The op is pure transport like `SendText`: no reconcile, re-render or persist, and `TmuxCmd.run` already logs the spawn at Debug (`PATTERN-spawn-observability`).

Tests:
- `io_test.go`: a table over `pasteSubmitArgv` pins the exact argv for a plain text with and without `submit`, a dash-leading text and a text ending in `;`; and one `PasteSubmit` row through `TmuxCmd.execHook` pins that a write issues exactly one tmux invocation (no `send-keys -l`) and that an unknown or hidden strand fails before any invocation.
- `paste_integration_test.go` (`//go:build tmux`, the package's `tmuxkit.Main` already isolates it): against a real tmux pane running `cat`, `PasteSubmit` with `submit` true delivers the text and the Enter (the pane shows the echoed line), `list-buffers -F '#{buffer_name}'` names no `lyx-inject-` buffer afterwards, and a buffer the test set beforehand with plain `set-buffer` is still the top of the default stack; with `submit` false the text sits on the prompt line with no newline.

Raise `internal/reedengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level tests.
Add a paragraph on the paste-submit op to `internal/reedengine/doc.go`'s pane-transport section.

**ImpactSummary:** New reed op and test file only; no existing caller changes.
