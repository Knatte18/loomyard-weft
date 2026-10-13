---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/reedengine/windowsize.go:186
    summary: the unseen-clear set-hook is targeted at a bare =session, which tmux rejects for set-hook, so the hook is never installed and own windows stay unseen after selection
    fix: In pinGeometryOptionsLocked pass exactSessionWindowTarget(e.SessionName()) instead of exactSessionTarget(e.SessionName()) as the -t argument of the set-hook call, since set-hook takes a pane target and needs the trailing colon. Update the expected argv in faketmux_test.go (the want slice near line 225) to the same helper so the fake test pins the colon form, and keep the doc.go and windowsize.go prose as they are.
  - id: F2
    severity: BLOCKING
    class: design
    location: internal/reedengine/window.go:68
    summary: OpenWindow chains remain-on-exit and the hidden mark onto the window name, so when an own window of the same name exists the options land on that older window and the new window reads remain-on-exit on
    fix: In OpenWindow create the window under a name no other window can hold (the requested name plus a per-call suffix), chain the two set-option calls onto that unique name, and chain a final rename-window of the same unique target to the requested name, all in the one multiplexer invocation. Keep the read-back by window id. Update the OpenWindow comment and the matching paragraph in doc.go (the chain-by-exact-name text) to say the chain targets a unique temporary name and renames last, and add the argv to the OpenWindow fake test that pins the chain.
  - id: F3
    severity: BLOCKING
    class: design
    location: internal/reedengine/ownwindow_integration_test.go:563
    summary: the pane-gone assertion uses display-message, which exits 0 for a missing pane under tmux 3.6, so the test fails even though the pane was killed
    fix: Replace the display-message check in the InvalidConfigStillRemovesAndReportsTheKey subtest with a check against the session-wide pane listing, for example slices.Contains of the list-panes -s -F '#{pane_id}' output (the listing used at line 405) against second.PaneID, and fail when the id is listed. Do the same for any other assertion that treats an error from display-message -t <pane> as proof a pane is gone.
---

### [BLOCKING] Unseen-clear hook is never installed

**Location:** internal/reedengine/windowsize.go:186

**Issue:** This is the cause of `TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks` in the publish failure.
`set-hook` takes a pane target, and tmux 3.6 answers a bare `=<session>` with `no such session: =<session>`; `=<session>:` works.
I reproduced both against a real tmux: the bare form exits 1, the colon form exits 0, and with the colon form selecting a window clears `@lyx_unseen` set on it.
The log shows the warning `failed to pin the unseen-clear hook ... no such session` on every boot and attach in the run, and the test then fails with `own window @lyx_unseen = "1" after it was selected`.
The fake test in `faketmux_test.go` asserts the same wrong target through the same helper, so it cannot catch this.

### [BLOCKING] OpenWindow options land on the wrong window when an own window shares the name

**Location:** internal/reedengine/window.go:68

**Issue:** This is the cause of `TestOwnWindow_RealTmux/OpenWindowOfTheSameNameLeavesTheOwnWindowAlone`.
The branch makes `findWindowLocked` skip own windows, so `OpenWindow("explore", ...)` now proceeds to create a second window named `explore` while the own window `explore` exists.
The chained `set-option -w -t =<session>:=explore remain-on-exit off` resolves the name to the older own window, so the new window keeps the session-wide `on`, the read-back fails and the call returns `ErrWindowReadBack`.
The chained option also changes the own window's `remain-on-exit` and adds `@lyx_hidden` to it, which breaks the guarantee the `findWindowLocked` change makes that an own window is left alone.
The doc paragraph that justifies chaining by exact name assumes only a concurrent same-name caller can collide, which the own-window feature makes routine.

### [BLOCKING] Pane-gone assertion cannot detect a missing pane

**Location:** internal/reedengine/ownwindow_integration_test.go:563

**Issue:** This is the cause of `TestOwnWindow_RealTmux/InvalidConfigStillRemovesAndReportsTheKey`.
`display-message -p -t %<id>` returns exit status 0 and an empty line for a pane that no longer exists (reproduced on tmux 3.6), so `err == nil` holds whether the pane was killed or not.
The log has no `could not enumerate panes` warning, so `killStrandPanes` ran; the removal behavior is not shown to be wrong, the assertion is.
The subtest's next check, `windowListed`, already reads the session-wide listing and is sound.
