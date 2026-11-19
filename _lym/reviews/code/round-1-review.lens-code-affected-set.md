```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each change in the diff, find everything outside the touched lines that reads, asserts or describes it, and flag what the diff leaves wrong.

**Affected set per change**

1. **`haltnote.Row.RoutedTo`, the `Render` fourth field, `parseRow` reading 3 or 4 fields.**
   - The writer `internal/loomcli/halt.go:73` `writeHaltNote` is covered: it copies `h.RoutedTo`.
   - The other `Parse` consumer, `internal/reportengine/stops.go:137` and `noteStop`, reads only Producer, State, Anomaly and Reason, never History, so it is unaffected by either row shape.
   - The tests `haltnote_test.go` `TestRenderParse` (new row) and `loomcli/halt_test.go` (route added) are covered.
   - Package doc `internal/haltnote/doc.go` names no row shape, so it is unaffected.
   - `PATTERN-sole-parsers` (`pattern/PATTERN-sole-parsers.md:76-80`) still holds: haltnote is still the sole renderer and parser and still imports only stdlib.
   - The source field's own doc, `internal/shedengine/status.go:106`, says RoutedTo is set on reroute entries and on Stuck entries. `internal/shedengine/reroute.go:59` sets it on `OutcomeReroute` entries. The haltnote doc comments do not match this (finding F1).
   - `internal/loomcli/halt.go:36-38` `writeHistoryRows` still writes three-field rows. It writes the crash-resume note (`entryobservation.go:182`), which is a separate format, and its doc stays accurate. Not a finding.

2. **`burlerengine.quotingHint` replaces `quotedSummaryHint` and gets a parse-error parameter.**
   - The only caller, `ParseReview` (`verdict.go:119`), is updated.
   - No reference to `quotedSummaryHint` remains anywhere in the repository.
   - No test outside `verdict.go` asserts the text "mapping values are not allowed" (searched), so no exact-message assertion breaks.
   - `ReviewGate` (`verdict.go:244-254`) passes `err.Error()` into the re-prompt findings, so the hint reaches the reviewer.
   - `TestParseReview` gains a covering row.

3. **Stencil `contracts/stencils/burler/burler-review-format.md`, the `summary` bullet widened to every value.**
   - The old rule that a quoted fragment followed by prose is invalid is kept (lines 50-52).
   - The old "just don't use literal quote characters" is kept as "leave the quote characters out".
   - No other stencil or spec repeats the old `summary` wording (searched).
   - No rule is dropped or weakened. The board asks that values containing `:` must be quoted, and line 46 carries that.

4. **`claudeengine.isTopBoxRule` accepts `… ─`.**
   - Its callers go through `inputBoxInterior`: `session.go` lines 99, 123, 172, 199 and 241, and `styledcapture.go:59`. All of them gain the behaviour; none needs an edit.
   - The doc comment is updated.
   - `claudeengine` and `shuttleengine` docs carry no top-rule description (searched).
   - Tests: two `TestInputBoxText` rows were added. The existing rows at `session_test.go:57` and `:67` (a label with no trailing glyph is no top rule) stay consistent.

5. **Commit messages.**
   - There are no `Pattern-Departure:` or `Loosened-Check:` trailers.
   - No assertion is loosened: `halt_test.go` `historyRows` now adds the trailing `\n` and the route, which tightens the assertion.
   - The directive declares no PATTERN changes, so there is nothing to land.

**Findings**

- **F1. LOW, consistency.** Location: `internal/haltnote/haltnote.go:21` (`routedPrefix` comment) and `:50-51` (`Row.RoutedTo` doc).
  - Summary: both comments describe the fourth field as the producer "a routine Stuck handed the run on to". Its source `shedengine.HistoryEntry.RoutedTo` is also set on `reroute` entries.
  - Evidence:
    - `internal/shedengine/reroute.go:59` writes `Outcome: OutcomeReroute, …, RoutedTo: def.OnReroute`.
    - `internal/shedengine/status.go:106` says "names the row a reroute entry sent the run to, or the row a Stuck entry's OnStuck edge bounced it to".
    - `internal/loomcli/halt.go:73` copies it through for every outcome.
  - The rendered `routed to` is right for reroute rows too. Only the two comments are narrower than the field they carry.

Tool calls:
1. Searched docs, specs and pattern files for the halt-note row shape and top-rule wording. My context held the definitions, not the docs that describe them.
2. Searched for other copies of the old `summary` quoting wording and for halt-note format docs. This checks stencil and doc staleness outside the diff.
3. Read `internal/haltnote/doc.go` and the claudeengine/shuttleengine doc mentions of the top rule. These are package docs that might describe the changed shapes.
4. Searched tests for exact assertions on "mapping values are not allowed". This checks whether the appended hint breaks an existing exact-message assertion.
