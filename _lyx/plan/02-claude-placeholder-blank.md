# Card 2 — claude-placeholder-blank

**Create:**
- `plan:internal/shuttleengine#StyledBoxReader` -> `type StyledBoxReader interface`
- `plan:internal/shuttleengine/claudeengine#Claude.PlainBoxCapture` -> `func (c *Claude) PlainBoxCapture(styled string) string`
- `plan:internal/shuttleengine/claudeengine#styledRun` -> `type styledRun struct`
- `plan:internal/shuttleengine/claudeengine#parseStyledLine` -> `func parseStyledLine(line string) []styledRun`
- `plan:internal/shuttleengine/claudeengine#TestPlainBoxCapture` -> `func TestPlainBoxCapture(t *testing.T)`
- `internal/shuttleengine/claudeengine/testdata/pane-placeholder-styled.txt`
- `internal/shuttleengine/claudeengine/testdata/pane-restored-prompt-styled.txt`

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.IdleSession`
- `internal/shuttleengine/claudeengine#TestIdleSession`
- `internal/shuttleengine/claudeengine#TestInputBoxText`
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#inputBoxInterior`
- `internal/shuttleengine/claudeengine#holdsOnlyEOLMark`
- `internal/shuttleengine/claudeengine#pastePlaceholderShape`
- `internal/shuttleengine/claudeengine#gateCaretMarker`
- `internal/shuttleengine/claudeengine#shellEOLMark`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`
- `internal/shuttleengine/claudeengine#readPaneFixture`

**Intent:** Decision D1, provider half: a styled capture of Claude's input box holding only its placeholder or prompt suggestion turns into a plain capture whose box reads empty.

`shuttleengine` declares, in `engine.go` beside `InputBoxReader`, the optional capability `StyledBoxReader` with the one method `PlainBoxCapture(styled string) string`: it takes a capture reed's `CapturePaneStyled` returned and answers the plain capture, every SGR and other escape sequence removed, with the cells of a box holding only the provider's placeholder blanked to spaces.
Its doc says a caller reads the box, the running turn and every other classification of one decision from the one plain capture it answers.

`Claude.PlainBoxCapture` splits the styled capture into lines and runs `parseStyledLine` over each: it walks the line, drops every CSI sequence (`ESC [` parameters and a final byte in `@`–`~`) and every OSC sequence (`ESC ]` up to `BEL` or `ESC \`), tracks the SGR faint attribute (parameter `2` sets it; `0`, `22` and an empty `ESC [ m` clear it; the other parameters leave it), and returns the line's text as `styledRun` values, each a text with its faint flag.
Joining the runs' texts gives the plain line, so for a capture with no placeholder the answer equals the escape-stripped capture.
The box is located on the plain lines with `inputBoxInterior`, exactly as `IdleSession` and `InputBoxText` locate it.
The box is blanked only when, once the caret marker, the side bars and whitespace are stripped from its interior lines and a lone `%` (`shellEOLMark`) standing as its own token is set aside as `holdsOnlyEOLMark` sets it aside, at least one non-blank cell remains, every remaining non-blank cell is faint, and the faint text joined does not match `pastePlaceholderShape`.
Blanking replaces each faint non-blank rune of the interior with a space; every other line, and every capture whose box fails the rule, is answered unblanked.
So unstyled text, a box mixing unstyled text with faint text, and a box drawing the collapsed-paste placeholder faint are never blanked, and a Claude release that restyles the placeholder reads as today's draft, fail-closed.
The style is pinned from the live capture of a fresh session taken at plan time: Claude draws the placeholder as `ESC[2m` text after the caret and an NBSP, and draws typed and restored text with no attribute.

The two fixtures hold the bottom rows of those live captures, written with a real ESC byte (0x1b) for every `\x1b` and a real NBSP (U+00A0) for every ` ` below, and `<rule>` standing for 100 `─` glyphs.
`testdata/pane-placeholder-styled.txt`, a fresh session's empty box with its suggestion:

```text
                                                                                 \x1b[38;2;153;153;153m◉ xhigh · /effort\x1b[39m
\x1b[38;2;136;136;136m<rule>
\x1b[39m❯ \x1b[2mTry "create a util logging.py that..."\x1b[0m
\x1b[38;2;136;136;136m<rule>
\x1b[39m  \x1b[38;2;255;193;7m⏵⏵ auto mode on\x1b[38;2;153;153;153m (shift+tab to cycle)\x1b[39m
```

`testdata/pane-restored-prompt-styled.txt`, the box right after an interrupt early in a turn put the prompt back, wrapped over three rows with the caret at its end:

```text
\x1b[38;2;136;136;136m<rule>
\x1b[39m❯ Please write a very long detailed essay about the history of the river Thames in London,
  covering the Roman period, the medieval bridges, the Victorian embankments, and the modern flood
  barrier, with at least twenty paragraphs
\x1b[38;2;136;136;136m<rule>
\x1b[39m  \x1b[38;2;255;193;7m⏵⏵ auto mode on\x1b[38;2;153;153;153m (shift+tab to cycle)\x1b[39m
```

The box readings are rows of the tables that already pin them: `TestIdleSession` and `TestInputBoxText` each gain rows whose capture is `PlainBoxCapture` of a styled capture, each asserting that table's own reading:
the placeholder fixture reads idle with empty text;
the restored-prompt fixture reads not idle with the prompt's text;
an unstyled typed draft reads not idle with its text;
typed text followed by faint text in one box reads not idle with both texts;
`[Pasted text #1 +4 lines]` drawn faint reads not idle with that text;
and faint placeholder text beside a lone unstyled `%` reads idle with empty text.
`TestPlainBoxCapture` is a new table test in `session_test.go` for the answer's own shape, which no test reads, each row comparing the answer with an expected plain capture:
the restored-prompt fixture's answer equals, line by line, the fixture with its escapes removed;
a capture with no box comes back escape-stripped and unchanged otherwise;
a faint line outside the box, such as a faint transcript line above it, stays in the answer;
an OSC 8 hyperlink sequence around box text is dropped from the answer, the text kept;
and two rows where faint placeholder text is followed in the box by text after `ESC[22m` and after a bare `ESC[m` come back unblanked, so a missed faint clear, which would blank typed text, fails them.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/shuttleengine/claudeengine` row rises from 113 to 114.

The comments that state the placeholder residual change with it: `Claude.IdleSession`'s doc keeps "a capture carries no styling" only for a plain capture and says a caller that reads through `PlainBoxCapture` sees a placeholder-only box as empty, under the bound above;
`settings.go`'s file header and the comment on `settingsDoc`'s `PromptSuggestionEnabled` field keep `promptSuggestionEnabled` false for shuttle-launched sessions and say the styled read is what lets a hand-added session's suggestion read as empty;
and `claudeengine/doc.go` gains the placeholder reading beside the lone-`%` sentence of its box paragraph.

**ImpactSummary:** Adds a shuttle capability nothing calls yet and Claude's realization of it; the comment edits change no behavior.
