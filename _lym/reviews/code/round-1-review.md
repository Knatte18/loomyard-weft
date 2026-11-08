---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: internal/reedengine/ownwindow_integration_test.go:385
    summary: The OS-bound subtest ResizeLaysTheSharedWindowOutEvenlyAgain is registered on Windows and returns early instead of being left out before t.Run
    fix: In TestOwnWindow_RealTmux wrap the whole t.Run("ResizeLaysTheSharedWindowOutEvenlyAgain", ...) call in `if runtime.GOOS != "windows" { ... }` and delete the early `if runtime.GOOS == "windows" { return }` inside its callback; the comment above it stays as the reason.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/reedengine/ownwindow.go:316
    summary: The zoom skip of the resize hook has no test, so removing the if-shell guard would pass the suite
    fix: In TestOwnWindow_RealTmux add a non-Windows subtest after ResizeLaysTheSharedWindowOutEvenlyAgain that opens a shared group of two, runs zoom with resize-pane -Z on the first pane, grows the window with resize-window -x, then asserts display of #{window_zoomed_flag} for the window is still 1 once the hook has run (query it after the resize command returns, since the hook is queued before the next command).
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/reedengine/ownwindow.go:306
    summary: The re-pin after a strand leaves a shared window is pinned by no test
    fix: Extend the existing leave subtest (LeavingASharedWindowRelaysItOut) or the new resize subtest so that, after a removal, the remaining window is resized and its panes are asserted to take equal widths; keep it one row of the existing test rather than a new test.
---

### [BLOCKING] OS-bound subtest registered on Windows

**Location:** internal/reedengine/ownwindow_integration_test.go:385

**Issue:** The subtest means something on one OS only, since psmux lacks `set-hook`, yet it is registered on Windows and returns at once with an empty pass.
`pattern/PATTERN-test-speed.md`, under "No test skips by design", requires a subtest bound to one OS to be left out before `t.Run`.
The file already follows that shape at line 146.
Every violation of a listed constraint is BLOCKING.

### [MEDIUM] Zoom skip of the resize hook is untested

**Location:** internal/reedengine/ownwindow.go:316

**Issue:** `evenLayoutResizeHook` guards on `#{?window_zoomed_flag,0,1}` so a resize never unzooms a window, and doc.go and the commit message claim it.
The only new test never zooms, so deleting the guard leaves every test green.

### [LOW] Re-pin on the leave path is untested

**Location:** internal/reedengine/ownwindow.go:306

**Issue:** `relayoutSharedWindowsLocked` pins each window's hook after the layout, and the docs state the hook is pinned again on a leave.
No test removes a strand and then resizes.
The hook is already present from the earlier layout, so the line changes behavior only after a `window_split` change, which keeps the severity LOW.

## Rejected

- code-tests, unpin leaves a group window's hook alone: `windowsize.go:127-128` runs `set-hook -u` with the shell window as `-t` and nothing else, so it cannot reach another window's option; the suspicion that it resolves to a current window has no ground.
- code-tests, failing `set-hook` never fails the step: a one-line logged branch of a private helper, and the trust-the-caller and test-economy rules give it no row.
- code-tests, layout quoting under `window_split=height`: the layout names are two fixed tmux tokens and the hook text is built by one expression.
- code-affected-set F-a, doc.go:343 wording: the sentence is true, since the shell window's unpin keeps no hook, and it sits where the shell window's hooks are described; a rewording changes nothing a reader acts on.
- Handler check of the focus questions: the hook is window-scoped through `set-hook -w`, its command takes no `-t` and runs against the firing window, `windowsize.go` unsets only the shell window's hook, both layout paths pin and the invalid `window_split` paths skip, and `codexengine` is untouched; none is a defect.
