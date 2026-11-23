```yaml
lens: code-correctness
outcome: reported
```

Task: the code-correctness lens. I traced each function and branch the diff adds or changes against concrete inputs.

**Per function or branch: the case I ran, and whether it is correct**

- `commentlint.countedText(raw)` with `"   - foo"`: indentation and the marker are stripped, the same as before the diff. Correct.
- `commentlint.isExemptFromLength(text, limit)` with a code span of exactly `limit` runes: exempt, as before with `LineLengthLimit`. Correct.
- `commentlint.TextReachesLimit` with `""`: returns false. With a heading at the limit: false, because it is exempt. With prose of 120 runes at limit 120: true. Correct, and the same as the old inline check in `findNewLongLines`.
- `commentlint.IsGenerated` with a CRLF marker line: `\s*$` under `(?m)` takes the `\r`, so it matches like the tool's old per-line scan. Correct.
- `commentlint.IsDirective` with `" lym:seam"` (leading space): `TrimSpace` gives `lym:seam`, which matches. Correct, and the same as the old inline call.
- `commentlint.IsProseLine` with `"\tcode"` (false), `"  - x"` (false), `" # H"` (false), `"foo"` with no space (true): this matches the old `isProseParagraph` logic on `raw` and `content`. Correct.
- `commentlint.IsSemanticBreak`:
  - With `after` holding no words, `words[0]` panics. The doc states the precondition, and every caller passes a non-blank line: lint paragraphs, written paragraph lines, and non-empty word slices from `splitAtBreaks` and `splitLongLine`. Correct.
  - With `before` given as raw text with its leading space (`split.go:246`): only the right edge is read. Correct.
- `godocreflow.planEdit`:
  - `/* */` group, or a group holding a directive: nil. Correct.
  - One-line group `"//"`: `TextReachesLimit("")` is false, so nil. Correct.
  - One-line long list item: `reflowParagraph` returns raws, `slices.Equal`, so nil. Correct.
  - Multi-line group with a blank line between paragraphs: the blank raw is kept verbatim. Correct.
  - `old` and `text` line counts equal the group's span. Correct.
- `godocreflow.reflowGroupLines` with raws `[" a", "", " b"]` and with raws `[" "]`: paragraphs flush at blank lines and whitespace-only lines are kept. Correct.
- `godocreflow.reflowParagraph` with a mixed prose and list-item paragraph: returned as written. A prose line with no space after `//` gains `" "`, which is gofmt style. Correct.
- `godocreflow.joinCommentLines` with a single raw: no indent and no newline. Correct.
- `godocreflow.layoutParagraph`:
  - One-line paragraph: the `written[:len-1]` loop is empty. Correct.
  - Word alignment between `writtenBreaks` and the `reflowText` output: `splitSentences`, `semicolonSplit` and `conjunctionSplit` drop only whitespace, so word counts line up. Correct.
- `godocreflow.splitAtBreaks` with a break at word 0 or at a line's end: these are excluded because `n` runs from 1 to `len-1`, so there are no empty pieces. Correct.
- `godocreflow.joinRejectedBreaks`:
  - A kept break stays accepted after later words are appended to its after-line, because `opensClause` only gains candidates within `clauseWords`.
  - A written break is always kept, and the lint's `carriesOver` excuses it through the word pair.
  - Result: no new wrapped-break findings. Correct.
- `godocreflow.splitLongLine`:
  - First accepted break already reaches the limit: it cuts there.
  - No accepted break: the line stays whole.
  - `cut` is at least 1, so the loop terminates.
  - The acceptance check uses `rest[n:]`, not the piece that will follow. This is wrong in a narrow case (finding 3).
  - It can leave a piece that still reaches the limit (finding 1).
- `godocreflow.reportLongLines`, two edits out of order where the first adds 2 lines: the sorted `lineShift` gives correct reflowed-file line numbers. It also reports lines it never reflowed (finding 2).
- `main`: if `WriteFile` fails, it exits 1 after printing the long lines, which is fine. In `-check` mode it reports reflowed-file line numbers, as the doc says. The tool's documented promise to name every line left whole is not kept (finding 2).

**Findings**

1. MEDIUM, design. `tools/godocreflow/split.go:313-347` (`splitLongLine`). A partial split turns a legacy long line the lint excuses into a new line-length finding.
   - When no accepted or written break brings every piece under the limit, the tool still splits the line.
   - The piece that still reaches the limit is a changed line, so `findNewLongLines` flags it (`internal/commentlint/linelength.go:68-80`: only an unchanged literal is excused).
   - Left whole, the line is unchanged and not flagged.
   - So the tool adds findings, against the goal of not worsening `lint-comments`. The summary's own evidence shows this: 141 line-length findings, each "a line that was already over the limit … the lint counts such a line as changed" (`_lym/landing/summary.md:34-38`).

2. LOW, design. `tools/godocreflow/reflow.go:153-158` and `tools/godocreflow/main.go:12-13`. The long-line report covers the wrong set of lines.
   - It counts every line of `newRaws` that reaches the limit, including list-item and indented-code lines that `reflowParagraph` returns untouched. Those lines are reported as "reflowed line(s) still reach" though the tool never changed them.
   - A long prose line with no break point, in a group that otherwise needs no change, makes `planEdit` return nil. That line is never named, though the package doc says "a line with none stays whole, and the run names it for a hand rewrite."

3. LOW, design. `tools/godocreflow/split.go:327-328`. `splitLongLine` can lose a usable split.
   - It judges a comma-and-conjunction break with `after` set to all remaining words. The output line after the cut can be shorter, because a later cut at a written break that falls inside the clause shortens it.
   - Example: written `…bar, and the store` / `was read …`. With `after` set to all remaining words, the check reads "was" and accepts the break. The final `joinRejectedBreaks` (`split.go:263`) then sees only `and the store`, rejects the break and rejoins the line.
   - The line reaches the limit again, and an earlier valid cut was given up.

Tool calls: none. Everything I needed was already in the inherited context.
