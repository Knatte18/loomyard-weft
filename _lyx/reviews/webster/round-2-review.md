---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/reedcli/smoke_selvage_keepalive_test.go:131
    summary: StaysPhysicallyBottomMostAcrossAddsAndRemoves still compares pane_top across every window of the session, but strands now live in own windows
    fix: In the StaysPhysicallyBottomMostAcrossAddsAndRemoves subtest, either scope requireBottomMost to the shell window's panes (list-panes without -s, targeting the Selvage pane's window) and assert Selvage is the only pane there after each add and remove, or replace the pane_top rule with that sole-pane assertion. Update the subtest's header comment to match, since the stacked-layout rule it cites no longer exists. listPaneLines in smoke_test.go lists with -s, so a new window-scoped helper may be needed.
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/reedengine/attachgeometry_integration_test.go:154
    summary: TestAttachGeometry_DegradedPathStillAttaches demands a 5-element argv, but with own-window strands AttachArgv(0, 0) now appends the select-window tail
    fix: Change the assertion to require the 5-element bare prefix (-L socket attach-session -t target) followed by the chained "; select-window -t <window>" tail (9 elements, per AttachArgv's contract in attach.go), or build the expected argv from the strand window id. Fix the test's comment ("bare form") accordingly. Cover the no-relevant-window case (bare 5 elements) only if a row for it does not exist yet.
  - id: F3
    severity: BLOCKING
    class: scope
    location: internal/reedengine/bar_integration_test.go:106
    summary: TestBar_LineOneButtonsFollowTheWindowZoomAndWaitMarks hardcodes range=window|1 for the extra window, but alpha and beta own windows now take indexes 0 and 1 and extra is window 3
    fix: Derive the extra window's index from tmux (display-message -p -t f.otherWindow '#{window_index}') and build the "range=window|<n>" expectations at lines 106 and 134 from it, instead of the literal 1. Re-check the later own-window-button assertions in the same test for other hardcoded indexes.
  - id: F4
    severity: BLOCKING
    class: scope
    location: internal/reedengine/bindings_integration_test.go:241
    summary: TestBindings_AltUpDownStepWindowsInIndexOrderPastHiddenOnes expects a step order that omits the alpha and beta own windows the fixture now creates
    fix: Rebuild the expected key sequence from the real window order. The fixture's alpha and beta strands, the agent strand, extra and overview are all separate windows now, so wrap-around from overview goes to the shell window first and each step passes through alpha and beta. Either list them in the sequence (and give currentWindow a name for each) or drop alpha and beta from this test's fixture.
  - id: F5
    severity: BLOCKING
    class: scope
    location: internal/reedengine/bindings_integration_test.go:277
    summary: TestBindings_StatusBarClicksSelectAndZoomByRangeType zooms alpha, which is now in its own window, so a shell click leaves alpha zoomed in a different window than the one the zoom flag is read from
    fix: Make the shell-click assertion meaningful under own windows. The shell button selects the shell window, which holds only Selvage, so zoom the Selvage pane (or drop the zoom half of the check) and read the zoom flag of the window the click lands on. Remove selectStrandsWindow(alpha) if it no longer selects what its name says. The window-button clicks below need their index/name expectations re-checked against the new window order.
  - id: F6
    severity: BLOCKING
    class: scope
    location: internal/reedengine/activity_integration_test.go:36
    summary: TestOperatorActive_ReadsAnAttachedClientsKeystroke selects the first strand's pane before attaching, but AttachArgv then selects the newest agent window, so the client sits on the other strand
    fix: Select the first strand's window after waitForClientAttached (select-window -t the window of strand.PaneID, then select-pane) so the keystroke lands on the strand OperatorActive is asked about, or swap which strand is added last. Keep the assertion that the other strand reads inactive.
  - id: F7
    severity: BLOCKING
    class: scope
    location: internal/reedengine/paste_integration_test.go:71
    summary: The sink strand pastes before its "stty -icanon" has run, so the 70002-byte paste is cut at the canonical 4095-byte line limit; own-window creation returns before the pane's shell runs the command
    fix: Make the sink command signal readiness after stty, for example "stty -icanon -echo; touch <readyFile>; cat > <sinkFile>", and wait on readyFile (poll with a deadline, as the test already does for the sink) before calling PasteSubmit(sink.GUID, longText, false). Reproduced under lyx gate test --tags tmux; the same test fails in the Publish log.
  - id: F8
    severity: BLOCKING
    class: design
    location: internal/reedengine/ownwindow_integration_test.go:731
    summary: The DeadPaneLeftInTheShellWindow subtest uses Cmd "exit", so Resume relaunches a pane that exits at once and the strand reads Live false
    fix: Give the leftover strand a relaunch command that stays up, for example set ResumeCmd to "sleep 300" in the Strand built by leftover (or pass a separate launch command) while the pane itself still runs "exit" to die. Then the post-Resume assertions (live, own window, fresh pane id, no second copy of the old pane) can hold. If the failure persists with a live relaunch command, treat it as a Resume/launchStrandLocked defect for a moved dead own-window pane and fix it in lifecycle.go or spawn.go.
---

### [BLOCKING] Selvage bottom-most smoke test still assumes one shared window

**Location:** internal/reedcli/smoke_selvage_keepalive_test.go:131
**Issue:** The subtest lists panes with `list-panes -s` and picks the largest `pane_top`.
Strands are placed in own windows now, so every window's first pane has top 0 and the rule is meaningless; the Publish log and a local `lyx gate test --tags tmux ./internal/reedcli` both fail it at every step.
The card that retired the stacked layout (card 2) left this test untouched.

### [BLOCKING] AttachArgv degraded-path test expects the pre-change argv

**Location:** internal/reedengine/attachgeometry_integration_test.go:154
**Issue:** The test asserts five elements; the engine returns nine, the bare argv plus `; select-window -t @2`, because strands in own windows make an agent window relevant.
`attach.go` documents that the selection is appended to every argv past the state read, so the code is right and the assertion is stale.

### [BLOCKING] Bar test hardcodes a window index

**Location:** internal/reedengine/bar_integration_test.go:106
**Issue:** The log shows alpha at window 0, beta at 1 and extra at 3, while the test expects `range=window|1` for extra at both line-one checks.
The rendering is correct; the expected strings were not updated when the fixture's strands moved into own windows.

### [BLOCKING] Alt step test ignores the fixture's own windows

**Location:** internal/reedengine/bindings_integration_test.go:241
**Issue:** The expected sequence starts with `extra` after `overview`, but the wrap goes to the shell window, then alpha and beta, before reaching `extra`.
Fails with "the Alt step never reached the extra window".

### [BLOCKING] Status bar click test zooms a pane outside the shell window

**Location:** internal/reedengine/bindings_integration_test.go:277
**Issue:** `alpha` lives in its own window, so a click on the shell button selects a different window and the zoom flag read afterwards is that window's, not alpha's.
The first assertion fails with zoom flag "0".

### [BLOCKING] OperatorActive test attaches onto the wrong strand

**Location:** internal/reedengine/activity_integration_test.go:36
**Issue:** The attach pre-flight selects the newest live agent window, which is `other`'s, overriding the `select-pane` made before attaching.
The keystroke therefore reaches `other`, and `strand` never reads active.

### [BLOCKING] Paste test races the sink's stty

**Location:** internal/reedengine/paste_integration_test.go:71
**Issue:** The file holds exactly 4095 bytes, the canonical tty line limit, which means the paste arrived before `stty -icanon` ran.
The test has no readiness signal, and the own-window launch path returns before the pane's shell has executed the command.

### [BLOCKING] Dead-pane move test cannot observe a live relaunch

**Location:** internal/reedengine/ownwindow_integration_test.go:731
**Issue:** The leftover strand's `Cmd` is `exit`, and `Resume` falls back to `Cmd` when `ResumeCmd` is empty (lifecycle.go), so the relaunched pane dies again and Status reports `Live:false`.
I did not rule out a product defect in relaunching a moved dead pane; the fix says to check that once the test's own flaw is removed.

## Other checks

The branch range was derived as `git diff $(git merge-base main HEAD)..HEAD`, since no `_lyx/loom/rework` round exists.
A grep of non-test sources found no `below-parent` or `top` outside the retired-anchor remap and refusal, and the retired config keys appear only in `RetiredKeys`.
