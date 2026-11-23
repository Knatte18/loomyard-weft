# godocreflow lays comments out by the comment lint's rules

Closes #183

`tools/godocreflow` no longer adds findings to `lym loom lint-comments`.
Run over trimmed Go files, it raised the wrapped-break findings from 624 to 916:
it skipped every comment block holding a heading or list, and it joined lines past the length limit.

## What changes

- `internal/commentlint` exports the rules it checks with: `IsGenerated`, `IsDirective`, `IsProseLine`, `IsSemanticBreak` and `TextReachesLimit`.
  `tools/godocreflow` imports them, so the tool and the lint cannot drift apart.
  The lint's own checks call the same functions; its behaviour is unchanged.
- `godocreflow` reflows the prose paragraphs of a block that also holds a heading, a list or indented code.
  Those non-prose paragraphs stay as written.
- A written break stays when the lint accepts it, or when the comment already broke between the same two words.
- A line whose text reaches the length limit is split at the latest accepted break that keeps the piece under it.
- A paragraph that no accepted break brings under the limit stays as written, since a changed long line would be a new finding.
- The run lists each prose line that still reaches the limit, by file and line number in the reflowed file, for a hand rewrite.
- The `-max-width` flag and its last-resort greedy word wrap are removed.
  The limit is `commentlint.LineLengthLimit`.

## Design decisions

- The tool imports the lint's rules instead of copying them, which reverses the old note in `internal/commentlint/doc.go` that the lint re-implemented the tool's kinds of skipped line.
- Keeping a paragraph as written over wrapping it by force trades completeness for never adding a finding.
  The long-line report is what makes that trade visible.

## To check by hand

- `latest-fitting-break-untested`: `splitLongLine` in `tools/godocreflow/split.go` takes the latest accepted break that fits; confirm `reflow_test.go` pins that choice.
- `layout-leaves-changed-lines-over-limit`: `layoutParagraph` in `tools/godocreflow/split.go` must keep a paragraph as written when its layout adds a long line; confirm the test covers it.
- `long-line-report-covers-wrong-lines`: `collectEdits` in `tools/godocreflow/reflow.go` must report every long prose line, at its line number after earlier edits shift lines.
- `long-line-report-untested`: confirm `reflow_test.go` pins the long-line report.
- Run `go run ./tools/godocreflow -check` over a package that holds headings or lists, then `lym loom lint-comments`, and confirm the findings do not rise.
  No before and after count was measured for this change.

## Merge danger

**Door:** two-way
**Blast radius:** godocreflow
