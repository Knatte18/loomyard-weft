# Card 6 — Manual window changes turn follow off; the follow button

**Create:**
- `plan:internal/reedengine#followCommandOption` -> `const followCommandOption = "@lyx_follow_command"`
- `plan:internal/reedengine#followClickCommand` -> `func followClickCommand(followJump bool) string`
- `plan:internal/reedengine#composeFollowCommand` -> `func composeFollowCommand(sh shell.Shell, socketPath, tmuxPath string) (string, bool)`
- `plan:internal/reedengine#followButtonFormat` -> `const followButtonFormat = "#[range=user|follow #{?#{!=:#{@lyx_follow},off},bg=green fg=black bold,bg=colour236 fg=white}] follow #[norange list=on default] "`

**Edit:**
- `internal/reedengine#bindingArgvs` -> `func bindingArgvs(switchPrev, switchNext string, followJump bool) [][]string`
- `internal/reedengine#statusClickBranches`
- `internal/reedengine#Engine.pinBindingsLocked`
- `internal/reedengine#Engine.switchBindingCommandsLocked` -> `func (e *Engine) switchBindingCommandsLocked() (prev, next, follow string)`
- `internal/reedengine#statusFormatSessions`
- `internal/reedengine#TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly`
- `internal/reedengine#TestComposeSwitchCommand`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly`
- `internal/reedengine#TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes`
- `internal/reedengine#TestBindings_StatusBarClicksSelectAndZoomByRangeType`
- `internal/reedengine#TestBindings_AltLeftRightRunTheSwitchCommandWithTheToldPaths`
- `internal/reedengine#TestBar_LineTwoListsSessionsInIDOrderAndTheBorderNamesStrands`
- `internal/reedengine/doc.go`
- `pattern/PATTERN-pane-binary-resolution.md`

**Uses:**
- `plan:internal/reedengine#followSessionOption`
- `plan:internal/reedcli#reedCLI.followCmd`
- `internal/reedengine#composeSwitchCommand`
- `internal/reedengine#newBindingsFixture`

**Intent:**
D5, D7 and D9: reed's manual window-change bindings write follow off, and one follow button on the session line toggles it.

Bindings (`bindings.go`), each `set-option` untargeted so it hits the clicking or key-pressing client's session:
- `M-Up` and `M-Down` start with `set-option @lyx_follow off` joined by `\;` before their window step, so the write lands also when the step wraps onto the current window.
- In `statusClickBranches`, the `shell`, `pane` and `window` branches each start with `set-option @lyx_follow off ; `, also for a click on the current window; the `session` branch is unchanged.
- `M-z`, `M-Left` and `M-Right` are unchanged.
- `bindingArgvs` appends a `follow` branch to the status click dispatch, on `#{==:#{mouse_status_range},follow}`, running `followClickCommand(followJump)`: `if-shell -F '#{==:#{@lyx_follow},off}' '<on>' 'set-option @lyx_follow off'`, where `<on>` is `set-option @lyx_follow on`, followed when `followJump` holds by ` ; run-shell -b "#{E:@lyx_follow_command}"`.
  A click from on sets off and changes no window; a click from off sets on and, when the jump is pinned, runs `lyx reed follow` in the background.
- `pinBindingsLocked` takes `prev, next, follow` from `switchBindingCommandsLocked`; with a non-empty `follow` it sets the server-global user option `@lyx_follow_command` (`set-option -g`) to it, otherwise unsets it (`set-option -gu`); a failure is logged at Warn and never fatal; it then passes `follow != ""` to `bindingArgvs`.
  Keeping the composed line in an option that run-shell expands with `E:` keeps the line out of tmux's nested command-string quoting.
- `switchBindingCommandsLocked` composes `follow` with `composeFollowCommand` from the same socket and tmux paths; every path that returns no switch commands returns no follow command either, through the same named Warns, so a click from off then only sets follow on and the next agent window takes the viewer (D7's fallback).

`composeFollowCommand`, in `panebin.go` beside `composeSwitchCommand`, returns the line `<exe> reed follow --socket <socket> --session #{q:session_name} --tmux <tmux>`: `exe` from `executablePath`, every pinned token quoted through `sh` and each `#` doubled, `#{q:session_name}` left as its own unquoted word for run-shell's expansion, so the session reaches the verb as one quoted argument; an unresolvable executable logs a named Warn and returns false.

Status bar (`bar.go`): `followButtonFormat` is a `range=user|follow` button labelled `follow`, lit (`bg=green fg=black bold`) while `@lyx_follow` is not `off` and in `unlitButtonStyle` while it is; `statusFormatSessions` places it right after the session loop, inside the left-aligned list part and before the `#[nolist align=right]` time, so it sits right of the session buttons and left of the time and is drawn at the offset the bindings fixture's `click` helper computes.
Its unlit style may be spelled through `unlitButtonStyle` by string concatenation.
`bar.go`'s and `bindings.go`'s header comments name `@lyx_follow` as the one option the bar shows that a Go decision reads.

D9: a psmux without `bind-key` or `run-shell` pins none of these, never writes off, and its sessions always follow; every binding stays non-fatal.

The bound: these bindings only ever write `off` (and the button `on`) to the clicking client's session, and the follow button runs one `lyx` verb whose effect card 5 bounds; a window change outside these bindings leaves follow on.

Tests:
`TestBindingArgvs_BindsTheFiveAltKeysAndTheStatusClickInTheRootTableOnly` passes the new argument, wants the `M-Up` and `M-Down` argvs with the leading `set-option @lyx_follow off \;`, checks the click dispatches on `follow` beside the four ranges with one more separator, that the `shell`, `pane` and `window` branch commands start with the off write and the `session` one does not, and that the follow branch runs `run-shell -b "#{E:@lyx_follow_command}"` with `followJump` true and holds no `run-shell` with it false.
`TestPinGeometryOptionsLocked`'s fixture tmux binary does not resolve, so `pinBindingsLocked` issues `set-option -gu @lyx_follow_command` as the last `set-option` of the geometry pins, and its subtests change accordingly:
`AllOptionsIssued` gains `{"set-option", "-gu", "@lyx_follow_command"}` as the last entry of its exact list;
`ShellPinsFollowTheOptionPins` wants the window-size pin and the two shell pins as the tail of the calls before that last unset, and the unset as the final call;
`SetHookErrorIsNonFatalWhenWatchdogOff` wants 9 `set-option` calls;
`BindingsAreIssuedAfterTheOptionPins` passes `bindingArgvs` the new argument, false, and asserts the unset is issued before the first `bind-key`;
and a new subtest `FailedFollowCommandWriteIsNonFatal` fails only the `set-option` carrying `@lyx_follow_command` through `answerFunc`, and asserts every binding is still issued and the Warn is logged.
`TestComposeSwitchCommand` gains a `follow` subtest pinning `composeFollowCommand`'s line against the same `#`-and-quote paths, and its unresolvable-executable row.
In the real-tmux bindings tests: `TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes` sets follow on before its steps and asserts it reads `off` after one; `TestBindings_StatusBarClicksSelectAndZoomByRangeType` sets follow on before each strand, shell and window click, a click on the current window's button included, and asserts `off` after it, and asserts the session click leaves the follow state of both sessions unchanged; `TestBindings_AltZTogglesStrandAndSelvageZoomInTheStrandsWindowOnly` and `TestBindings_AltLeftRightRunTheSwitchCommandWithTheToldPaths` assert their keys leave `@lyx_follow` as it was.
`TestBindings_StatusBarClicksSelectAndZoomByRangeType`, which pins the click dispatch one range type at a time, gains the `follow` range, before its session click: with follow on, a click on the line-2 `follow` button sets it off, keeps the current window and runs no recorder; a second click sets it on and the recorder records `reed follow --socket <socket path> --session <session> --tmux <absolute tmux path>`; with `@lyx_follow_command` unset on the server and follow off, a click sets follow on, keeps the current window and runs no recorder (D7's fallback); and, after the session click, with the client on a session named with a space, a quote and a `#` and follow set off there, a click records that name intact as the `--session` value.
The test creates that session itself with `new-session -d -s <name>` and moves the client there with `switch-client`, so `newBindingsFixture` stays as it is.
`TestBar_LineTwoListsSessionsInIDOrderAndTheBorderNamesStrands` asserts the expanded line 2 carries the lit follow button with `@lyx_follow` unset and the unlit one with it `off`, after the last session button and before the time.

`doc.go`'s bindings paragraph states which bindings write off, the follow button's toggle and its fallback, and the bound that changes outside reed's bindings leave follow on; its status-bar paragraph adds the button.
`PATTERN-pane-binary-resolution.md`'s list of bindings that run `lyx` gains the follow button.

**ImpactSummary:** `bindingArgvs` and `switchBindingCommandsLocked` each have one caller, `pinBindingsLocked`, on this card; the status format and the click branches are pinned by the listed Tier-1 and real-tmux tests, which gain rows and no new top-level test.

**Commit:** 6: reed: manual window changes turn follow off; one follow button toggles it
