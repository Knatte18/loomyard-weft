```yaml
lens: code-scope
outcome: reported
```

Task: trace each change in the diff to a goal point of the decision record, and each goal point to the changes that build it (code-scope lens).

**Decisions.** The record is a decision record alone. Its `## Goal` holds the board entry. It has no `Added after Discussion` entries.

- **G1:** The idle check finds the input box by the `❯` line and the borders around it. This holds whatever text the top border carries and however few `─` are left beside it.
- **G2a:** A start prompt still held after its retries raises a notice to the parent naming the strand.
- **G2b:** A respawn on the same row does not repeat the failure.

**Changes, each traced to a decision:**

- **C1:** `isTopBoxRule` takes any line that starts and ends with `─`, and its doc comment says so (`internal/shuttleengine/claudeengine/session.go:335-346`). Traces to G1, and through the fixed cause to G2b.
- **C2:** A new `TestInputBoxText` row for the #196 capture (`session_test.go`). Forced by C1.
- **C3:** `sendStartPrompt` now splits the busy check and calls the new `noticeStartPromptHeld` (`internal/shuttleengine/run.go:933-969`). Traces to G2a.
- **C4:** Doc updates in `doc.go:70`, the `Runner.notifier` field comment (`run.go:55-57`) and the `SetNotifier` doc (`run.go:96`). Forced by C3.
- **C5:** `TestStartup_SendFailuresTearDown` gains `wantHeldNotice` and the notice assertions (`startup_test.go`). Forced by C3.
- No change traces to nothing, and no commit carries a `Pattern-Departure:` trailer that would need checking.

**Decisions, each with the changes that build it:**

- **G1:** Built by C1 and C2, partly (finding 1 below).
- **G2a:** Built by C3, C4 and C5.
- **G2b:** Built by C1, since the respawn hit the same check. The respawn path itself is unchanged, and the summary says so in its design decisions. The board entry leaves the hub's own sighting "for the run to judge", and the summary answers that it cannot be settled from the code. That claim is no scope gap.

**Findings**

1. LOW, scope.
   - **Location:** `internal/shuttleengine/claudeengine/session.go:343-351` (`isTopBoxRule`).
   - **Summary:** G1 asks for the top border to be found "however few `─` are left beside it". The new check covers one glyph on each side, the cut label `… ─` and a plain rule. It still refuses a labelled border with no leading glyph and no ellipsis, such as ` &lt;label&gt; ─`.
   - **Why it may matter:** A pane one column narrower than the #196 shape could draw that, before Claude starts cutting the label. G1 then holds only for the shapes the diff names, not for "however few" glyphs.
   - **Uncertainty:** I could not confirm that Claude ever draws this shape.
   - **Fix:** Either pin from a capture that Claude never draws the shape, or accept a line whose last token is a single `─` after a label.
   - **Evidence:** `session.go:345-350`; `_lym/discussion/decision-record.md:42`.

I made no tool calls; everything came from the context I inherited.
