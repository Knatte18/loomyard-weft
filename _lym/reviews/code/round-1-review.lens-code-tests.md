```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, the test that pins it, or none.

**Per behaviour**

1. **A discussed open task's Slug cell stacks `[Sketch](decision-<slug>.md)` under the design-linked slug.** `TestRenderSlugCellLinksTheDecisionRecord`, row T1, `render_test.go:932`. T1 sits in Ready, so the row is read against the Ready column order, Task then Slug then Recipe. Reverting `slugColumnCell` to `slugCell` fails the row. The assertion runs on both `README.md` and `project-root.md`.
2. **A discussed done entry gets the Sketch link in the Done table.** The same test, row T3, `render_test.go:934`. Done has the Entry and Slug columns and no Recipe. T3 has no body, so this also pins the Sketch link under a plain, unlinked slug code span.
3. **An entry not flagged discussed renders as before.** The same test, row T2, `render_test.go:933`. The substring ends at `| darn |` right after the slug, so any appended `<br>[Sketch]...` breaks the match.
4. **The `cellBreak` join keeps the row valid with and without a design link.** T1 covers the body case and T3 the no-body case. Both rows are single-line, and the `|`-delimited prefix match fixes the cell boundary.
5. **The link target is `DecisionFileName(slug)`.** The T1 and T3 rows hard-code `decision-sketched.md` and `decision-landed.md`. A wrong prefix or suffix would fail them.
6. **Every table that lists a discussed entry shows the mark.** Running, layers, Independent, Draft and Notes have no row of their own. All of them render through the one `b.table` call site (`render.go:449`), so Ready and Done reach that same code, and a row per table would duplicate coverage. This is not a finding.
7. **A project's facts line keeps plain `slugCell`.** The behaviour is unchanged, since `render.go:368` still calls `slugCell`, so nothing new needs a row. No test asserts it, but routing the facts line through `slugColumnCell` is not a regression the diff makes. This is not a finding.
8. **`doc.go` and header comment prose.** These are static prose, so they are exempt.

**Helpers and redundancy**

- `stringPtr` is at `layer_test.go:285`, `renderTime` at `render_test.go:25` and `readmeTypes` at `render_test.go:244`. All resolve, so the test compiles.
- The other `Discussed` uses are in `decision_test.go` and `replay_test.go`. They cover the store, the payload rule and replay, not rendering.
- No existing test asserts a `Sketch` link or a discussed Slug cell, so `TestRenderSlugCellLinksTheDecisionRecord` duplicates nothing.

**Findings:** none. Every behaviour the diff adds or changes has a failing row if broken, or shares a code path with one that does.

**Tool calls (1 of 6)**
- `grep` for `stringPtr`, `renderTime`, `readmeTypes` and `Discussed` across `internal/boardengine/*_test.go`: my context lacked their definitions and any existing render test that covers a discussed row.
