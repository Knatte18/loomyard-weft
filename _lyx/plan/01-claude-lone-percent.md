# Card 1 — claude-lone-percent

**Create:**
- `plan:internal/shuttleengine/claudeengine#boxInteriorText` -> `func boxInteriorText(interior []string) string`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `pattern/PATTERN-pane-text-injection.md`

**Intent:** Claude's input-box reading treats a box whose only content is a lone `%` as empty (decisions D1 and D2, issue #549).
`boxInteriorText` is the join `InputBoxText` does today (caret and side bars removed, each line trimmed, non-blank lines joined by one space), returning `""` when that joined text is exactly `%`.
`InputBoxText` returns `boxInteriorText(interior)` with `ok` true.
`IdleSession` keeps its blank-line loop, and a line that loop finds non-blank no longer returns false at once: it returns whether `boxInteriorText(interior)` is empty, so a lone `%` reads idle and any other content, `%` beside other text included, still reads as a draft.
The `%` knowledge stays in `claudeengine` (`PATTERN-shuttle-provider-seam`); `shuttleengine` gains nothing.
Both callers of the box reading, shuttle's pre-write guard and the startup sends (color send and skill-load send) read through `IdleSession` and `InputBoxText`, so the third acceptance criterion follows from the two rows below with no change in `shuttleengine`.
What the change lets through: an operator draft of exactly `%` typed outside `operator_quiet_s` reads as empty, and a paste can merge with it; the session idle probe's fallback for a strand with no run reads the box with no operator-activity hold, so there a lone `%` reads idle with no operator check.
The operator-activity hold still runs before every write behind the pre-write guard, and nothing but the one character `%` alone is let through.
`pattern/PATTERN-pane-text-injection.md` gains, under the pre-write guard bullet and in the same commit, one line (semantic line breaks, no fixed wrap): the provider's box reading defines "empty", and Claude's reads a lone `%`, the shell's end-of-line mark left on screen, as empty; an operator draft of exactly a lone `%`, typed outside `operator_quiet_s`, reads as empty and a write can merge with it.
The entry line in `PATTERN.md` is not touched.
Rows, added to the existing tables and not as new top-level tests: `TestIdleSession` gains a lone `%` after the caret (true), `%` beside other text (false) and a single other character such as `#` (false); `TestInputBoxText` gains the same three captures (`""`, `"% done"`, `"#"`, all `ok` true).

**ImpactSummary:** Two Claude methods and one new private helper in `claudeengine`; every caller of the box reading (pre-write guard, startup sends, orch watcher) sees a lone `%` as empty, no other behavior changes.
