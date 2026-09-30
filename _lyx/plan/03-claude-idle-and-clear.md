# Card 3 — claude-idle-and-clear

**Create:**
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/shuttleengine/claudeengine/testdata/pane-idle-empty.txt`
- `internal/shuttleengine/claudeengine/testdata/pane-idle-draft.txt`
- `internal/shuttleengine/claudeengine/testdata/pane-permission-prompt.txt`
- `internal/shuttleengine/claudeengine/testdata/pane-turn-running.txt`

**Uses:**
- `internal/shuttleengine/claudeengine/startup.go`
- `internal/shuttleengine/claudeengine/usage.go`
- `internal/shuttleengine/session.go`

**Intent:**
Complete Claude's `SessionCycler` with the idle-session probe (D9) and the clear-session choreography (D13), and pin the implementation with `var _ shuttleengine.SessionCycler = (*Claude)(nil)`.

`func (c *Claude) IdleSession(capture string) bool` returns true only when all of these hold:

- No turn is in progress: the capture does not contain Claude's running-turn hint ("esc to interrupt", matched in `normalizeCapture` form like `startup.go`'s needles).
- The input box is on screen: the last line carrying `gateCaretMarker` has a horizontal-rule line (runs of `─`, optionally with box corners or side bars) somewhere above it and another below it, with only box-interior lines between the two rules.
- The box is empty: every line between the two rules, once the caret, side bars and whitespace are stripped, is blank.

A permission prompt or any other select dialog draws its caret on an option line that is not framed by the input box's rules, so it fails the second check;
a draft fails the third.
A capture carries no styling, so Claude's greyed prompt suggestion inside an empty box is indistinguishable from a draft and classifies as not idle.
That fails closed — the watcher skips the poll and retries — and the doc comment says so;
card 14's smoke test would stall at its cycle wait if suggestions block cycling in practice.

`func (c *Claude) ClearSessionSequence() []shuttleengine.PaneInput` returns `/clear` typed and submitted, with no leading Escape.
The doc comment gives the reason: the caller has just proved the box empty, so there is nothing to clear, and an Escape landing right after another Escape opens Claude's rewind menu instead.

The pane fixtures are plain-text captures of a Claude Code TUI in the shape `capture-pane -p` returns, following the existing startup fixtures' convention: transcribe them from a live capture when one is available, otherwise from the current TUI's rendering.
`pane-idle-empty.txt` shows transcript lines above an empty input box and the `ReadyFooterFixture` footer below it;
`pane-idle-draft.txt` is the same box holding typed text;
`pane-permission-prompt.txt` shows a tool permission dialog with its numbered options and caret;
`pane-turn-running.txt` shows a turn in progress with its running-turn hint above an empty box.

`session_test.go` covers `IdleSession` over each fixture (idle only for the empty box), a capture with no input box at all, and a transcript line quoting "esc to interrupt" above an empty box — which classifies as not idle, a fail-closed residual the doc comment names.
It also pins `ClearSessionSequence` to exactly one text step, `/clear`, with `Submit` set.
