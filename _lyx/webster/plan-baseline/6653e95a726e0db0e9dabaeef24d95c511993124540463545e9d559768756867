# Card 9 — claudeengine takes the paste path and drops the leading Escape

**Create:**
- `plan:internal/shuttleengine/claudeengine#pasteSubmitsOnEnter` -> `const pasteSubmitsOnEnter = true`
- `plan:internal/shuttleengine/claudeengine#PasteCollapseMinChars` -> `const PasteCollapseMinChars = 800`
- `plan:internal/shuttleengine/claudeengine#Claude.PasteSubmits` -> `func (c *Claude) PasteSubmits() bool`
- `plan:internal/shuttleengine/claudeengine#Claude.OwnPastePlaceholder` -> `func (c *Claude) OwnPastePlaceholder(boxText, text string) bool`
- `pattern/PATTERN-pane-text-injection.md`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.ComposeSend`
- `internal/shuttleengine/claudeengine#Claude.TypeSequence`
- `internal/shuttleengine/claudeengine#Claude.ClearSessionSequence`
- `internal/shuttleengine/claudeengine#Claude.CompactSessionSequence`
- `internal/shuttleengine/claudeengine#Claude.ReloadPluginsSequence`
- `internal/shuttleengine/claudeengine#Claude.ColorSequence`
- `internal/shuttleengine/claudeengine#pastePlaceholderShape`
- `internal/shuttleengine/claudeengine#TestComposeSend`
- `internal/shuttleengine/claudeengine#TestClearSessionSequence`
- `internal/shuttleengine/claudeengine#TestCompactSessionSequence`
- `internal/shuttleengine/claudeengine#TestReloadPluginsSequence`
- `internal/shuttleengine/claudeengine#TestColorSequence_TypesTheColorCommandForEveryPaletteColor`
- `internal/shuttleengine/claudeengine#TestPastePlaceholder`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/template.yaml`
- `PATTERN.md`

**Delete:**
- `internal/shuttleengine/claudeengine#composeSendSettleMS`
- `internal/shuttleengine/claudeengine#Claude.PastePlaceholder`

**Uses:**
- `plan:internal/shuttleengine#TextPaster`

**Intent:**
Move Claude onto the paste path (decisions D2 and D6), which makes every text lyx types into a Claude pane go through reed's paste-submit behind shuttle's guard, and record that invariant in PATTERN (D12).

`Claude` implements `shuttleengine.TextPaster`, asserted in `session.go`'s compile-time block beside `InputBoxReader` and `InputBoxClearer`.
`PasteSubmits` answers `pasteSubmitsOnEnter`, true under assumption A7 (an Enter right after the bracketed paste end submits); card 10's live check pins it, and if that check finds otherwise this one constant flips to false, which makes shuttle paste without the Enter and send the first Enter as a guarded one.
`OwnPastePlaceholder(boxText, text)` answers true only when `boxText`, trimmed, matches `pastePlaceholderShape` as a whole, `text` has at least `PasteCollapseMinChars` characters (Claude Code's collapse threshold, exported so card 10's live check in another package sizes its texts from it and pins it), and, when the placeholder shows `+<m> lines`, `m` equals the number of newlines in `text`.
`pastePlaceholderShape` gains a capture group for the line count; `PastePlaceholder`, which had no other caller after card 6, is deleted, and `TestPastePlaceholder` becomes the table over `OwnPastePlaceholder` (a short text's placeholder is refused, a line-count mismatch is refused, a quoted placeholder inside other words is refused).

The leading Escape and its 300 ms settle go (D6: prompt suggestion is off in every shuttle-launched session, so the Escape guarded nothing and could complete a double-Esc or interrupt a turn): `ComposeSend` returns the text step and the Enter, `TypeSequence` the text step alone, and `composeSendSettleMS` is deleted.
`ClearSessionSequence`, `CompactSessionSequence`, `ReloadPluginsSequence` and `ColorSequence` drop the `submitSettleMS` pause between their text and their Enter, since the paste path submits them in one tmux invocation; their doc comments say so.
`ModelSwitchSequence` and `InterruptSequence` are explicit key deliveries outside this task and keep their shape.
`internal/shuttleengine/template.yaml`'s `submit_settle_ms` comment drops "the pause before the Enter of the /clear, /compact, /reload-plugins and /model commands" and states it paces the two box reads before the Enter of the no-Enter paste variant and of a typing engine, and the pause before `/model`'s Enter.

PATTERN: add under "Agents and prompts", after `PATTERN-shell-mechanics-seam`, this entry line verbatim:

```markdown
- `PATTERN-pane-text-injection` — Typing text into an agent pane: only reed's paste-submit, behind shuttle's pre-write guard or, for name repair, reed's activity check; no Enter of lyx's own into a box not holding exactly the sent text. — [background](pattern/PATTERN-pane-text-injection.md)
```

`pattern/PATTERN-pane-text-injection.md` holds the background: the guard's three readings (readiness, operator activity within `operator_quiet_s`, an empty and readable box), the one-invocation paste-submit and why it closes the merge window, the exact-match and placeholder rule for a guarded Enter, the pending-send record, the explicit key deliveries outside it (`Runner.Interrupt`, `Runner.Inject`, `InterruptSequence`, `ModelSwitchSequence`, the trust-prompt dismissal), and that no caller may reintroduce `send-keys -l` plus a blind Enter.

Tests: `TestComposeSend` pins both sequences with no `Escape` key and no `SettleMS`; the clear, compact, reload and color sequence tests pin the text step with no pause followed by the Enter.
Update `internal/shuttleengine/claudeengine/doc.go` for the paste path and the dropped Escape.

**ImpactSummary:** Claude's typing sequences lose a key and a pause, one method is replaced; shuttle is the only caller and already takes the paste path.
