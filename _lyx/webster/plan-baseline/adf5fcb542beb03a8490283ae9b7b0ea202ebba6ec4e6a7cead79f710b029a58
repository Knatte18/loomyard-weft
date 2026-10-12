# Card 3 — paste-stdin-buffer

**Edit:**
- `internal/reedengine#TmuxCmd`
- `internal/reedengine#TmuxCmd.run`
- `internal/reedengine#TmuxCmd.output`
- `internal/reedengine#pasteSubmitArgv` -> `func pasteSubmitArgv(paneID, buffer string, submit bool) []string`
- `internal/reedengine#Engine.pasteSubmitLocked`
- `internal/reedengine#Engine.PasteSubmit`
- `internal/reedengine/doc.go`
- `internal/reedengine#fakeTmux`
- `internal/reedengine#fakeTmux.exec` -> `func (f *fakeTmux) exec(capture bool, stdin []byte, args ...string) (string, error)`
- `internal/reedengine#installFakeTmuxOn`
- `internal/reedengine#TestSwitchClientVia_IssuesOnlyTheThreeCommands`
- `internal/reedengine#TestPasteSubmitArgv`
- `internal/reedengine#TestPasteSubmit`
- `internal/reedengine#TestRepairNames_SessionName`
- `internal/reedengine#TestPasteSubmit_DeliversTextAndLeavesTheBufferStackAlone`

**Create:**
- `internal/reedengine#TmuxCmd.runStdin`
- `internal/reedengine#fakeTmux.StdinFor`

**Intent:**
D3: paste-submit loads its buffer from the tmux client's standard input, so a text of any length pastes.

Runner (`overlay.go`):

- `TmuxCmd.execHook`'s type becomes `func(capture bool, stdin []byte, args ...string) (string, error)`; its comment says `stdin` is the bytes a `runStdin` call writes to the command's standard input, nil for `run` and `output`, passed beside the subcommand argv rather than inside it.
- `TmuxCmd.run` and `TmuxCmd.output` pass a nil `stdin` to the hook; nothing else in them changes.
- New `TmuxCmd.runStdin(stdin []byte, args ...string) error`: `run` with `stdin` written to the command's standard input (`cmd.Stdin = bytes.NewReader(stdin)`), stdout discarded and stderr folded into the error through `wrapTmuxError`.
  It logs its full args at Debug as `logger.Debug("tmux", "args", fullArgs)`, exactly as `run` and `output` do (`PATTERN-spawn-observability`), and never logs the stdin bytes.
  With `execHook` set it calls `execHook(false, stdin, args...)` instead.

Paste (`io.go`):

- `pasteSubmitArgv` drops its `text` parameter and returns `load-buffer -b <buffer> -`, `;`, `paste-buffer -p -d -b <buffer> -t <paneID>`, then `;` and `send-keys -t <paneID> Enter` when `submit` is true.
  The trailing-`;` escape and the `--` guard go: the text is no longer an argv element.
  Its doc comment says the text arrives on standard input and so never adds, ends or alters a command in the list.
- `pasteSubmitLocked` runs `e.tmux.runStdin([]byte(text), pasteSubmitArgv(paneID, buffer, submit)...)`; the failed-invocation `delete-buffer` of the named buffer stays as it is.
- `Engine.PasteSubmit`'s doc comment drops "A text longer than tmux's message limit fails the whole invocation and pastes nothing" and says the text of any length is loaded from standard input.
- `internal/reedengine/doc.go`'s paste-submit paragraph says the buffer is loaded with `load-buffer -` from the tmux client's standard input in the same invocation, replaces the message-limit sentence with "A text of any length pastes whole, since tmux's command size limit applies to the command list and not to a buffer read from standard input", and keeps the failed-invocation `delete-buffer` sentence.

Bound: the text reaches tmux only as stdin bytes of `load-buffer -`, never as an argv element, so no text can add, end or alter a command in the list; size is unbounded by design.
The two callers are unchanged: shuttle's send behind its pre-write guard, and reed's name repair with its fixed rename line.
The paste and the Enter stay in one tmux invocation (`PATTERN-pane-text-injection`).

Fake (`faketmux_test.go`):

- `fakeTmux` gains a `stdins [][]byte` field, appended beside `calls` and `captures` on every call (nil for `run` and `output` calls).
- `fakeTmux.exec` takes the new `stdin` parameter and records it; on the forward-to-real path it calls `real.runStdin(stdin, args...)` when `stdin` is non-nil, else `real.output` or `real.run` as today.
- `installFakeTmuxOn` keeps assigning `cmd.execHook = f.exec`, now of the new type.
- New `fakeTmux.StdinFor(verb)` returns, per call to `verb` in order, the stdin bytes as a string.
- `TestSwitchClientVia_IssuesOnlyTheThreeCommands`'s own hook closure takes the new `stdin []byte` parameter and ignores it.

Tests:

- `TestPasteSubmitArgv`: the rows pin the new list, `load-buffer -b buf -` then the paste, with and without the trailing Enter.
  The dash-leading and trailing-semicolon rows are deleted, since the function no longer receives the text.
- `TestPasteSubmit`: the scripted failure answers `load-buffer` instead of `set-buffer`, `pastes` reads `fake.ArgvFor("load-buffer")` (the buffer name stays at index 2), and the success path also asserts `fake.StdinFor("load-buffer")` equals `[]string{"hello"}`.
  The table gains no text row: no code path reads the text after this card, and the dash-leading, semicolon-ending text is pinned in a real pane by the integration test below.
- `TestRepairNames_SessionName`: `pastes` reads `fake.ArgvFor("load-buffer")`, the "want none" and "exactly one" messages name `load-buffer`, and the rename-text check reads `fake.StdinFor("load-buffer")` equal to `[]string{"/rename tc:s:worker"}` instead of searching the argv.
- `TestPasteSubmit_DeliversTextAndLeavesTheBufferStackAlone` (`paste_integration_test.go`, tier `tmux`) gains a long paste before its `list-buffers` check.
  It adds a second strand whose command is `stty -icanon -echo; cat > <file>`, with `<file>` under `t.TempDir()`: the pane's terminal must be non-canonical because a canonical tty line holds at most 4095 bytes, so a plain `cat` would drop most of a long line.
  It pastes, with submit false, one line of at least 70 000 bytes that starts with `-` and ends with `;`, polls the file until its size reaches the text's length or a 10-second deadline passes, and asserts the file's content equals the text.
  The existing `list-buffers` and `show-buffer` checks then also cover the long paste's buffer.

**ImpactSummary:** Reed only: `pasteSubmitArgv`'s two callers (`pasteSubmitLocked`, `TestPasteSubmitArgv`) and `execHook`'s two setters (`installFakeTmuxOn`, the switch test's closure) are on this card; `PasteSubmit`'s signature and its shuttle and name-repair callers are unchanged.

**Commit:** 3: reed: paste-submit loads its buffer from stdin, so a text of any length pastes
