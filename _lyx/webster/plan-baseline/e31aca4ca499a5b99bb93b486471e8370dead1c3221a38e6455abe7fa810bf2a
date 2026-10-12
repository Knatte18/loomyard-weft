# Card 2 — pane readings: running turn, rewind picker and interrupt settle

**Create:**
- `internal/shuttleengine#InterruptGuard`
- `internal/shuttleengine#RewindPickerReader`
- `internal/shuttleengine/claudeengine#interruptSettle`
- `internal/shuttleengine/claudeengine#rewindPickerNeedle`
- `internal/shuttleengine/claudeengine#Claude.RunningTurn`
- `internal/shuttleengine/claudeengine#Claude.RewindPicker`
- `internal/shuttleengine/claudeengine#Claude.InterruptSettle`
- `internal/shuttleengine/claudeengine#TestInterruptGuardReadings`
- `internal/shuttleengine/claudeengine/testdata/pane-rewind-picker.txt`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.StrayInput`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine#TestStartup_Classification`
- `internal/shuttleengine/claudeengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#runningTurnNeedle`
- `internal/shuttleengine/claudeengine#readPaneFixture`

**Intent:**
D3, D4 and D8 detection: the provider capabilities the guards read, provider-neutral in `shuttleengine` and Claude-specific in `claudeengine` (`PATTERN-shuttle-provider-seam`).

`internal/shuttleengine/engine.go` gains two optional capabilities beside `Engine`, declared like `InputBoxReader` and `StrayInputReader`, with godoc on each method:

- `InterruptGuard` with `RunningTurn(capture string) bool`, whether the capture shows a turn in progress, and `InterruptSettle() time.Duration`, how long after an interrupt key the pane needs before it is read again, longer than the provider's double-interrupt window.
- `RewindPickerReader` with `RewindPicker(capture string) bool`, whether the capture shows the provider's rewind picker in place of its input box.

No caller reads them until card 4.

In `internal/shuttleengine/claudeengine/session.go`, `Claude` implements both; the `var` block of compile-time assertions gains `shuttleengine.InterruptGuard` and `shuttleengine.RewindPickerReader`.

- `RunningTurn` is `strings.Contains(normalizeCapture(capture), runningTurnNeedle)`, the needle `IdleSession` and `StrayInput` already match.
  Both of those switch to calling `c.RunningTurn(capture)`, so there is one running-turn reading; `Startup` is left alone.
  The godoc states D3's bound: a transcript line quoting the hint anywhere in the capture reads as a running turn.
- `InterruptSettle` returns the constant `interruptSettle`, 2 seconds, whose comment states the assumption that Claude Code's double-Esc window is under 2 seconds.
- `RewindPicker` is true when the normalized capture contains `rewindPickerNeedle`, the picker's subtitle "Restore the code and/or conversation to the point before…" in `normalizeCapture` form, and `inputBoxInterior(capture)` finds no input box.
  The no-box condition keeps a transcript that quotes the subtitle above a live input box from reading as the picker, so the close never sends an Esc onto an idle prompt for a quote.
  The godoc states the bound from the record's Open risks: a release that rewords the subtitle leaves the picker undetected.

The fixture `internal/shuttleengine/claudeengine/testdata/pane-rewind-picker.txt` is a live `tmux capture-pane -p` of Claude Code 2.1.296's rewind picker, opened by a double Esc after one finished turn, taken by the plan writer on 2026-10-12, with the banner and the blank rows above the picker trimmed.
Write it with exactly this content; the line of `▔` is cut to 60 glyphs:

```text
❯ Reply with the single word ok.

● ok

✻ Crunched for 4s · done 19:33

▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔▔
   Rewind

   Restore the code and/or conversation to the point before…

     Reply with the single word ok.
     No code changes

   ❯ (current)



   Enter to continue · Esc to cancel
```

Tests:

- New `TestInterruptGuardReadings` in `session_test.go`, one table over captures with both readings per row:
  - the picker fixture: picker true, running false;
  - `pane-turn-running.txt`: running true, picker false;
  - `pane-idle-empty.txt`: both false;
  - a capture quoting the picker subtitle in the transcript above an empty input box drawn with the `rule` lines `TestIdleSession` uses: picker false, the row that fails if the no-box condition is dropped;
  - a capture quoting "esc to interrupt" above an empty box: running true, pinning D3's stated bound;
  - `pane-permission-prompt.txt`: both false, a dialog with no input box that is not the picker.
  
  After the table it asserts `New().InterruptSettle() == 2*time.Second`.
- `TestInputBoxText` gains the row "rewind picker has no box": the picker fixture answers `("", false)`, so the picker-close re-check in card 4 reads a still-open picker as no input box.
- `TestStartup_Classification` gains the row "rewind picker reads ready": the picker fixture classifies `StartupReady`, because its caret marks it ready.
  The guards in card 4 run after the ready-agent-pane check, and this row fails if a change makes that check refuse the picker before the close can see it.
- `cmd/lyx/testdata/test-budget.yaml`: raise `internal/shuttleengine/claudeengine` from 108 to 109 for `TestInterruptGuardReadings`.

`claudeengine`'s `doc.go` gains a paragraph beside the one on `StrayInput`: the running-turn reading shared by `IdleSession`, `StrayInput` and `RunningTurn`, the picker reading and its no-box condition, and the 2-second settle.

**ImpactSummary:** Two additive interfaces in `shuttleengine` with no reader yet; in `claudeengine`, `IdleSession` and `StrayInput` keep their answers because `RunningTurn` is the same needle test, so `TestIdleSession` and `TestStrayInput` hold unchanged.

**Commit:** 2: Read a running turn and the rewind picker from a Claude pane
