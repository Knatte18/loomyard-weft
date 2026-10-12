# Card 7 — the status bar gives each own window a colored button marked unseen until selected

**Create:**
- `plan:internal/reedengine#ownWindowButtonFormat` -> `const ownWindowButtonFormat = ""`
- `plan:internal/reedengine#windowWaitFormat` -> `const windowWaitFormat = ""`
- `plan:internal/reedengine#unseenClearHookName` -> `const unseenClearHookName = "session-window-changed"`
- `plan:internal/reedengine#unseenClearHookCommand` -> `const unseenClearHookCommand = "set-option -w -u @lyx_unseen"`

**Edit:**
- `internal/reedengine/bar.go`
- `internal/reedengine#Engine.pinGeometryOptionsLocked`
- `internal/reedengine/doc.go`
- `internal/reedengine#TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks`
- `internal/reedengine#TestPinBootOptionsLocked_InstallsTheSignalEntry`
- `internal/reedengine#TestPinGeometryOptionsLocked`
- `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`
- `internal/reedengine#TestAttachArgv_PreflightOnAKnownGoodSession`

**Uses:**
- `plan:internal/reedengine#unseenWindowOption`
- `internal/reedengine#ownWindowOption`
- `internal/reedengine#Engine.AttachArgv`

**Intent:**
D4's announcement: a new own window shows up on line 1 of the status bar, in its segment's color, carrying its strands' wait marks and styled unseen until the operator first selects it, while nothing moves the operator.
Every option here is display only (`PATTERN-wait-mark-display-only`): no Go decision reads `@lyx_unseen` or `@strand_color`.

In `internal/reedengine/bar.go` (the two new formats sit beside the existing ones, and their real values are the format strings below; the `Create` heads only declare the names):

- The file comment says the bar reads strands through `@strand`, `@strand_color` and `@lyx_strands`, and own windows through the window options `@lyx_own`, `@strand_color` and `@lyx_unseen`, and that a window carrying `@lyx_hidden` still gets no button.
- `windowWaitFormat` expands, for the window the `#{W:}` loop is on, the `#{P:}` loop of its panes' `strandWaitFormat`, so every waiting strand of an own window shows ` ⏳<label> <minutes>m` on its button.
- `ownWindowButtonFormat` is one window range, `#[range=window|#{window_index} ...]`, labelled ` #{window_name}` followed by `windowWaitFormat`:
  lit (`list=focus`, the window's `@strand_color` as background, black, bold) while the window is current;
  otherwise unlit in its color on dark grey (`bg=colour236 fg=<@strand_color>`), with `underscore` added while `@lyx_unseen` is set.
  The color falls back to white when the window has no `@strand_color`, as `strandColorFormat` does for panes.
- `statusFormatButtons` gives a window carrying `@lyx_own` the own-window button, and keeps the existing branches for `@lyx_strands`, `@lyx_hidden` and every other window; a window carrying `@lyx_own` gets its button whether or not the session has another window, since window 0 always exists beside it.
- The status click's existing `window` branch (`select-window -t=`) selects an own window from its button, so no binding changes.

`pinGeometryOptionsLocked`, everywhere but Windows, pins the session hook `set-hook -t =<session> session-window-changed 'set-option -w -u @lyx_unseen'`, after the navigation bindings and non-fatal like them.
Boot and every attach pre-flight run this function, so the hook is pinned at boot and re-pinned on each attach, like the navigation bindings.
Bound: the hook unsets one window option, on the window the client moved to, and nothing else.
On Windows no hook is pinned and card 4 never sets `@lyx_unseen`, so no window reads unseen forever.
Its doc comment names the hook.

`internal/reedengine/doc.go`'s status-bar paragraph describes the own-window button, its unseen styling and the hook.

Tests:

- `TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks` gains a step: an own-window strand added in the review segment gives line 1 a `range=window|<index>` button labelled with the window's name in `colour208` with `underscore`; a wait mark set on its strand shows on that button; after `select-window` to it, the expanded line shows it lit and the window's `@lyx_unseen` reads empty; after selecting the strands' window again, the button is unlit without `underscore`.
  The existing assertion that the `@lyx_hidden` window gets no button still holds with the own window present.
- `TestPinBootOptionsLocked_InstallsTheSignalEntry` filters its `set-hook` list to the `window-resized` hook name, so the resize-hook assertions stay exact, and asserts in both steps one `set-hook -t =<session> session-window-changed` call carrying `unseenClearHookCommand`.
- `TestPinGeometryOptionsLocked`: `WatchdogOnPinsGeometryOptionsOnly` asserts that no `set-hook` call names `windowResizedHookName`, and one `session-window-changed` call carrying `unseenClearHookCommand`, in place of no `set-hook` call at all, and its comment says the function installs no resize hook;
  `SetHookErrorIsNonFatalWhenWatchdogOff` wants two `set-hook` calls, the unseen-clear pin and the resize-hook unset, both still attempted while each errors.
- `TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv` replaces `fake.mustNotCall("set-hook")` with an assertion that no `set-hook` call names `windowResizedHookName`, since the degraded rows past the pin still issue the unseen-clear hook.
  Its doc comment changes from "issue no set-hook call at all" to "install no resize-pin hook".
- `TestAttachArgv_PreflightOnAKnownGoodSession` filters the `set-hook` calls to those naming `windowResizedHookName` before the first-index and `-u` checks, and asserts exactly one `session-window-changed` call carrying `unseenClearHookCommand`.

**ImpactSummary:** Display-only change to bar.go's formats plus one non-fatal session hook in the shared pin path that boot and attach run; no Go decision reads the new options.

**Commit:** 7: The status bar shows own windows in their segment color, unseen until selected
