---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/burlerengine/verdict.go#quotingHint
    summary: "The quoted-summary branch runs first and its regex also matches a valid, fully quoted summary, so a colon error elsewhere in the header gets the wrong hint and the new colon hint never shows."
    fix: "In internal/burlerengine/verdict.go, change malformedQuotedSummary so it matches within one line only: use a space-or-tab class in place of the whitespace class before and after the quoted fragment, so it can no longer run on into the next line's first character, and let the fragment skip backslash-escaped quote characters. In TestParseReview (internal/burlerengine/verdict_test.go) add a row whose summary is fully quoted and whose fix value holds an unquoted colon-space, expecting the colon hint's own text (a value holding), and add a row whose quoted summary fragment is followed by prose holding a colon-space, expecting the fragment hint's own text (not a quoted fragment followed by trailing prose). On the existing bad yaml row, also assert that the error lacks the text a value holding."
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: consistency
    location: internal/haltnote/haltnote.go:21
    summary: "The routedPrefix comment and the Row.RoutedTo doc describe the routed field as a routine Stuck hand-off, but reroute history entries carry RoutedTo too and render the same suffix."
    fix: "In internal/haltnote/haltnote.go, reword the routedPrefix comment to say the fourth field names the producer the row's step routed the run on to, by a routed Stuck or a reroute, and reword the second line of the Row.RoutedTo doc the same way, so neither names Stuck alone."
    origin: lens:code-affected-set
---

### [MEDIUM] The colon hint is pre-empted by a fragment regex that matches valid quoted summaries

**Location:** internal/burlerengine/verdict.go#quotingHint

**Issue:** `quotingHint` checks `malformedQuotedSummary` (`verdict.go:266`, `(?m)^\s*summary:\s*"[^"]*"\s*\S`) before the new colon branch.
In RE2 `\s` matches a newline, so a valid `summary: "the gate fails"` followed by any next line matches through that line's first character.
The pattern also matches the stencil's own escaped form `summary: "\"capital\" is ..."`, since `"[^"]*"` closes at the escaped quote.
Once any summary in the header is quoted, which the new stencil rule tells reviewers to do, every colon parse error in a `fix` or another field is reported with the fragment hint.
The reviewer is then re-prompted with advice about a summary that is already correct, and the colon hint this diff adds is never shown.
No test row has both a quoted summary and a colon error, so the order is unpinned (the code-tests lens's gap, folded in here).

### [LOW] The routed-field comments name Stuck alone

**Location:** internal/haltnote/haltnote.go:21

**Issue:** `routedPrefix`'s comment says the fourth field is "the producer a routine Stuck handed the run on to", and `Row.RoutedTo`'s doc says it tells a routine Stuck hand-off from a stop.
`shedengine.HistoryEntry.RoutedTo` is also set on `reroute` entries (`internal/shedengine/reroute.go:59`, documented at `internal/shedengine/status.go:106`), and `writeHaltNote` copies it for every outcome, so a reroute row renders `/ routed to <producer>` as well.
The rendered output is right for both outcomes; only the two comments are narrower than the field.

## Rejected

- assist F1 (a draft line ending in `… ─` taken as the top rule): the walk up from the caret starts above the last caret line, so the case needs a draft holding a second literal `❯` on a later line; that contrived input already defeats the existing `───…─` labelled-rule branch the same way, and the halted pane's real shape is pinned by the new rows.
- assist F2 (stencil narrows `:` to `: `): a colon without a following space is valid in a plain YAML value, and the format's own `location: path/to/file.go:42` relies on that; quoting on `: ` is the precise rule the board entry's parse failure asks for.
- code-tests LOW (branch order unpinned): the same defect as F1, merged into it; its test rows are in F1's fix.

## Focus checks

- `routed to` on reroute entries: correct output for both a routed Stuck and a reroute; only the comments are narrow (F2).
- `parseRow`: three-field rows still parse, four-field rows round-trip with `Render`, and `internal/reportengine` reads only the note's producer, state, reason and anomaly, so either shape is safe.
- `unquotedColonError`: the text is the pinned yaml.v3 v3.0.1 scanner's own (`scannerc.go:1411`); the precedence defect is F1.
- Stencil: the escape rule and examples are valid YAML, and the old fragment rule and its two remedies are kept.
- Re-prompt scope: `ReviewGateEntry` at the review base already re-prompts the reviewer with the parse error; the claudeengine fix restores the delivery the commit message names, so it is licensed by the board entry's first bullet of each issue.
- `isTopBoxRule`: the new branch is reached only above the caret, and the added rows cover the cut label with and without its trailing glyph.
