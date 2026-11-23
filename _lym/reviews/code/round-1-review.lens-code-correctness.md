```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. For each function and branch the diff adds or changes, I ran one concrete case through the code.

**Cases run**

1. `isTopBoxRule` (`internal/shuttleengine/claudeengine/session.go:343-352`), new prefix/suffix branch.
   - Input: `"─ dop:models-torque-drag-adapter:plan-burler-review ─"`. It trims, starts and ends with `─`, and returns true. Correct.
   - Input: a plain rule `"────"`. It returns true on the first branch, the same as the old `isBoxRule` fallback. Correct.
   - Input: `" ly:x:code-burler… ─"`. The first branch fails and the `cutLabelRuleEnd` branch returns true, as before. Correct.
   - Input: `" ly:x:code-burler…"` with no glyph after it. Both branches fail and `isBoxRule` is false, so it returns false. Correct.
   - Input: `"╭───╮"`. The first branch fails because the line opens with `╭`; `isBoxRule` returns true. Correct.
   - Input: an empty or blank line. The trim yields `""`, which has neither prefix nor suffix, and `isBoxRule` is false. Correct.
   - In `inputBoxInterior` (`session.go:311-317`), the walk up from the caret stops at the nearest match. The real top rule sits directly above `❯`, so a widened match on a transcript line further up is never reached when the box is present. The bottom rule still uses the unchanged `isBoxRule`, so a draft line shaped like `─ x ─` below the caret still never closes the box. Correct.
2. `sendStartPrompt` (`internal/shuttleengine/run.go:933-949`):
   - When the error is not busy (nil, `ErrSubmissionNotLanded`, a failed pause wait), the first `if` returns err unchanged. Correct.
   - A busy error at retry 0 while the deadline has not passed logs and loops. Correct.
   - A busy error at retry 2 sends the notice and returns the same err, so the caller's `errors.Is(err, ErrSessionBusy)` and `abandonStartup` path are unchanged. Correct.
   - A busy error at retry 0 with the deadline already passed sends the notice and returns err. "Read busy on every send" holds, since only one send was made. Correct.
3. `noticeStartPromptHeld` (`run.go:955-969`):
   - With a nil notifier it returns, so no notice is sent. Correct.
   - On an interactive run it returns. Correct.
   - With an empty `StrandName` it falls back to `StrandGUID`, the same as `noticeLoadHold` at `loadturn.go:343-346`. Correct.
   - With a reason holding a newline, `delimitedAgentText` swaps the control characters for spaces and cuts the text to 200 runes, so the notice stays one line. Correct.
   - When notify returns an error, it is logged at Warn and the start's error is unaffected. Correct.
   - `QuietHold` is not checked. That matches `Spec.QuietHold`'s contract, "skips the held-turn parent notice alone" (`spec.go:85`), and the skill-load notice, which ignores it too. Correct.
4. `loadSkillsThenPrompt` (`run.go:907-912`), the caller, unchanged: the busy error is not `ErrSubmissionNotLanded`, so it reaches `keepFailureCapture` and `abandonStartup`. The notice's "untyped and the strand torn down" holds. Correct.
5. The test assertion block (`startup_test.go` diff lines 133-145) asserts the exact count of notices naming `strand-1` and the total count. It cannot pass on an extra or a missing notice. Correct.

**Findings**

- NIT, consistency, `internal/shuttleengine/run.go:932`. The `sendStartPrompt` doc says the last busy hold returns "after a notice to the parent naming the strand" without qualification. The notice is skipped for an interactive run and for a runner with no notifier (`run.go:957`), so the contract overstates when the notice goes out. Evidence: `run.go:932`, `run.go:956-959`.

No other correctness defect was found.

Tool calls: none. Everything needed was already in the inherited context.
