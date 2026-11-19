```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens — for each behaviour the diff adds or changes, name the test row that would fail if that behaviour broke.

**Per behaviour, the test that pins it**

1. **`quotingHint` adds the colon hint when a plain value holds `: `.** Pinned by `internal/burlerengine/verdict_test.go` `TestParseReview`, row "unquoted colon in a value appends the quoting hint". That row's `errSubstr` appears only in the new hint, so removing the branch fails the row.
2. **The quoted-summary-fragment hint still fires.** Pinned by `TestParseReview`, row "malformed quoted summary appends a targeted hint" (verdict_test.go:217-231). That content has no `: `, so the colon branch cannot produce the shared substring "must be ONE double-quoted string covering the whole value". Removing the regex branch fails the row.
3. **The quoted-summary hint wins over the colon hint when a header triggers both.** No row pins this. Every quoted-summary row lacks `: `, so swapping the two `if` blocks in `quotingHint` passes every test.
4. **The colon hint stays off for parse errors without a colon.** No row pins this. The "bad yaml" row (verdict_test.go:206-213) asserts only `not valid YAML` and never that the hint is absent.
5. **`Render` adds ` / routed to <producer>` when `RoutedTo` is set.** Pinned by `internal/haltnote/haltnote_test.go` `TestRenderParse`, row "a routine stuck row names the producer it handed the run on to" (`wantContain`).
6. **`Render` adds no suffix when `RoutedTo` is empty.** Pinned by the `blockedGolden` byte comparison in the same test (haltnote_test.go:13-22, 64-65).
7. **`parseRow` reads the fourth field and strips `routed to `.** Pinned by the routed row's `Parse` round-trip with `reflect.DeepEqual` (haltnote_test.go:70-76).
8. **`parseRow` still reads older three-field rows.** Pinned by the `blockedGolden` round-trip and by "an older note's parent fabric status section…".
9. **`parseRow` handles a fourth field without the `routed to ` prefix, or a field that itself holds ` / `.** No row. This is out of this lens's scope: Render never writes that shape, so the correctness lens judges it.
10. **`writeHaltNote` copies `HistoryEntry.RoutedTo` into the note row.** Pinned by `internal/loomcli/halt_test.go` `TestWriteHaltNote_NamesAnomalyProducerReasonAndHistory`. Its Webster row expects ` / routed to Plan-Write\n`, and the Preflight row expects a bare `\n` ending, which pins that an unrouted row gets no suffix.
11. **`isTopBoxRule` accepts a line ending in `… ─`.** Pinned by `internal/shuttleengine/claudeengine/session_test.go` `TestInputBoxText`, row "a label too long for the pane cuts the top rule to the label and one glyph". The capture has no other rule above the caret, so removing the branch gives `ok=false` and the row fails.
12. **`isTopBoxRule` rejects a cut label with no glyph after it.** Pinned by `TestInputBoxText`, row "a cut label with no rule glyph after it is no top rule". Loosening the suffix to `…` fails the row.
13. **The gate re-prompt now carries the colon hint.** Pinned indirectly: `ReviewGate`'s `Findings` embeds `err.Error()` (verdict.go:250-252), and item 1 pins that text.
14. **Stencil quoting rules in `burler-review-format.md`.** Static prose, exempt.

**Findings**

- **LOW, design.** Location: `internal/burlerengine/verdict.go:274` (`quotingHint`). No test pins either half of the new branch order:
  - that the quoted-summary hint wins when both hints apply;
  - that the colon hint stays off for other YAML errors.

  Evidence: verdict_test.go:206-213 (the "bad yaml" row asserts only `not valid YAML`) and verdict_test.go:215-231 (the quoted-summary row has no `: `, and its `errSubstr` is shared by both hints). Fix: in `TestParseReview`, add a row whose summary is a quoted fragment with trailing prose holding `: `, asserting the quoted-summary hint's own text, such as "not a quoted fragment followed by trailing prose". On the "bad yaml" row, also assert that the error does not contain `a value holding`.

Tool calls:
1. `sed`/`grep` over `internal/haltnote/haltnote_test.go:1-100` and `internal/burlerengine/verdict_test.go:200-232`. My context held only the diff hunks of these tests, not the bodies and existing rows needed to tell whether a broken behaviour would fail a row.
