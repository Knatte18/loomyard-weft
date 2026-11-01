# Card 9 — button-click-binding

**Create:**
- `plan:internal/reedengine#buttonCommandOption` -> `const buttonCommandOption = "@lym_button_command"`
- `plan:internal/reedengine#composeButtonCommand` -> `func composeButtonCommand(sh shell.Shell, executablePath func() (string, error), socketPath, tmuxPath string) (string, bool)`
- `plan:internal/reedengine#TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` -> `func TestBindings_ButtonClickRunsTheVerbInTheRunDirectory(t *testing.T)`

**Edit:**
- `internal/reedengine#bindingArgvs` -> `func bindingArgvs(switchPrev, switchNext string, followJump, buttonClick bool) [][]string`
- `internal/reedengine#Engine.switchBindingCommandsLocked` -> `func (e *Engine) switchBindingCommandsLocked() (prev, next, follow, button string)`
- `internal/reedengine#Engine.pinBindingsLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine#bindKeyArgvCount`
- `internal/reedengine#TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`
- `internal/reedengine#TestComposeSwitchCommand`
- `internal/toolfloor/doc.go`

**Uses:**
- `internal/reedengine#composeFollowCommand`
- `plan:internal/reedengine#buttonDirOption`
- `plan:internal/reedengine#buttonRangePrefix`
- `plan:internal/reedengine#Engine.pinButtonRowLocked`
- `plan:internal/reedengine#ButtonContext`
- `plan:internal/reedengine#ButtonBeam`
- `plan:internal/reedengine#ButtonPlaceTask`
- `internal/reedengine#newBindingsFixture`
- `internal/reedengine#bindingsFixture.click`
- `internal/reedengine#clickArgv`
- `internal/reedengine#waitUntil`
- `internal/reedengine#mustTmuxOutput`
- `internal/shell#Shell`

**Intent:**
A left click on a button range runs `lym reed button` in the session's run directory (D7).

- `composeButtonCommand` returns the pinned click line, composed like `composeFollowCommand`: `sh.Chain(sh.ChangeDirToWord("#{@lym_button_dir}"), sh.DiscardOutput(<words>), sh.ExitSuccess())`, so the verb starts in the session's run directory, and the words the absolute path of this binary through `executablePath`, `reed button`, `--socket` and `--tmux` with the told paths quoted, `--session #{q:session_name}`, `--range #{q:mouse_status_range}` and `--client #{q:client_name}`; every pinned token is quoted once and has each `#` doubled, the three `#{q:...}` words and the directory word `#{@lym_button_dir}` left for the click's expansion; the directory word needs no `q:`, since the render pins it already quoted in reed's dialect (card 6), which a POSIX-quoting `q:` would break under pwsh, and tmux substitutes an option's value without expanding it again (probe: a value holding `#{session_name}` reached the shell literally).
  The directory change rides in the line because tmux 3.6 `run-shell` format-expands its command but not its `-c` start directory (probe: with a global `@g` of `/usr`, `run-shell -c '#{@g}'` ran in the home directory, while `cd` with the directory in the command reached `/usr`), so D7's `run-shell -b -c '#{@lym_button_dir}'` would start the verb in the server's directory.
  A failed change, a run directory removed after the render, leaves the verb in the server's directory, where it shows the gone-directory message (card 8).
  The discarded output and the forced zero status keep view mode shut on every path, a removed or replaced pinned binary, a panic and a kill signal included.
  An unresolvable executable logs a named Warn and returns false.
- `switchBindingCommandsLocked` also returns the button line, all four empty when the socket path, the tmux path or this binary does not resolve, as now.
- `pinBindingsLocked` sets `buttonCommandOption` globally when the line resolves and unsets it otherwise, each failure logged at Warn, beside the follow line, so a stale line never outlives the binary that pinned it.
- `bindingArgvs` takes `buttonClick`: when true the click dispatch ends, after the follow branch, with `\;` and `if-shell -F '#{m:lymbtn*,#{mouse_status_range}}' 'run-shell -b "#{E:@lym_button_command}"'`, so the one global binding resolves the per-session command through the session's own options; when false no button branch is bound.
  The follow branch, its range and look are unchanged.
  The file's header comment names the button branch.
- Bound, from D7: a click runs only the line the binding names, which runs only the spec entry at the clicked index; a click whose pinned binary is gone runs nothing, logs nothing and shows no message, until a reed op of another build not older than the pinned one re-pins the bindings (`bindingsStale`).

`reedengine/doc.go`'s binding paragraphs add the button branch, its pinned line in `@lym_button_command` and the Bound above.
`toolfloor/doc.go`'s list of tmux features found gains the `#{m:}` match modifier and `confirm-before -p`, both at or below the floor.

Tests:
- `TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` (`tier0`) passes `buttonClick` on every call: with it the click argv ends with the button branch exactly as above and joins its branches with one more escaped separator; without it the argv holds no `lymbtn`.
- `bindKeyArgvCount` and the `BindingsAreIssuedAfterTheOptionPins` call in `TestPinGeometryOptionsLocked` (`tier0`) pass `false`; `wantOptions` gains `set-option -gu @lym_button_command` after the follow-command unset, the pin counts grow by one, `ShellPinsFollowTheOptionPins`'s `wantLast` becomes the follow-command unset, the button-command unset and the build-stamp unset, with its length check and messages following, and `FailedFollowCommandWriteIsNonFatal` is unchanged.
- `TestAttachArgv_PreflightOnAKnownGoodSession` (`tier0`): `wantSetOptionCalls` is `16 + wantShellPins`.
- `TestComposeSwitchCommand` (`tier0`) gains a `button` subtest pinning the exact POSIX line for the `it's#here` paths and an unresolvable-executable subtest.
- `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory` (`tier2 && linux`, in `bindings_integration_test.go`) uses `newBindingsFixture` with the executable swapped for a script that records `pwd`, its arguments and whether `LYM_STRAND_NAME` and `LYM_PARENT` are set, then exits 3.
  It sets `e.cfg.Buttons` to the plain entry `HALT` and the `window: agents` entry `AGENT`, and `geom.Buttons` to a stub with `Place` `ButtonPlaceTask` whose `Beam` returns a temp directory as `Anchor`, so the run directory is the beam's anchor and not the session's (card 6), then re-runs the geometry pins under the op lock:
  line 3 shows `AGENT` in a strand's own window and not in the shell window, and `HALT` in both;
  a click on `HALT` on row 2 records that temp directory as `pwd`, which pins the line's directory change, neither strand-name variable set, since the server's environment was stripped of them at spawn (D7), and the arguments `reed button --socket <socket> --tmux <abs tmux> --session <session> --range lymbtn0 --client <client>`;
  after the script's exit 3, the active pane's `#{pane_in_mode}` reads `0`;
  a click on `follow` still toggles the follow state.

**ImpactSummary:** two re-signed package-private functions whose only callers are `pinBindingsLocked` and the bindings and pin tests, all listed; one more global option pinned beside the follow line; the click binding gains one branch.

**Commit:** 9: Bind a click on a status-bar button to lym reed button
