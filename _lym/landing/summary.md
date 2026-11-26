# Read an unindented continuation line as part of its record bullet

A decision record bullet whose second line is not indented was cut at its first line.
`discussionparser.SectionBullets` continued a bullet only through indented lines,
so a criterion such as

```markdown
- D6: the client attaches
only inside the area.
```

was read as `D6: the client attaches`.
Raddle carries criteria by exact text, so a part could carry only the cut text:
the whole bullet, joined to one line, failed the raddle-propose gate as `criterion-unknown`.
The acceptance coverage check in `CheckDecisionIDs` had the same rule through `bulletLines`,
so a decision id named only on such a line counted as uncovered.

## Change

Both readers now share one rule, `continuesItem` in `internal/discussionparser/decisionids.go`.
A line right after a bullet's line continues the bullet when it is indented,
or when it is unindented, as markdown reads a lazy continuation,
unless it is an ATX heading, a thematic break or a blockquote, or opens an HTML block that can interrupt a paragraph.
A blank line, the next bullet, a fenced block or a carry-over block still ends the bullet.
A spaced thematic break such as `* * *` is no longer read as a bullet.
`SplicePart` writes a carried bullet's continuation lines indented, so a spliced part record reads back the same text.

`contracts/specs/loom-discussion-spec.md` now states where a bullet ends.

## Design decisions

- The parser reads the line as markdown does, rather than the record format check refusing it.
  A stored record that already holds such a line then reads as it renders, with no rewrite.
- The HTML-block exception keeps a carry-over block's `<!-- lym:carry-over … -->` marker, or any other comment,
  from joining the bullet above it.
- A `#` or `<` line ends a bullet only when markdown would end it there,
  so a continuation opening with an issue number or a placeholder such as `<slug>` stays in the bullet.

## Tests

- `TestSectionBullets` gains the reported case, rows for lines that only resemble a heading or an HTML block,
  and rows for lines that end a bullet.
- `TestSplicePart` reads back a criterion with an unindented continuation line from a spliced record.
- `TestCheckDecisionIDs` covers a decision id named only on an unindented continuation line.

## To check by hand

- `continuesItem` in `internal/discussionparser/decisionids.go`:
  it excluded too many lines opening with `#` or `<` from a bullet.
  Check that the ATX heading and HTML block patterns now match only lines that end a paragraph in markdown.
- `TestSectionBullets` in `internal/discussionparser/partsplice_test.go`:
  the HTML-block and blockquote exclusions had no test pinning them.
  Check that the comment and blockquote rows do.
- The `## Merge danger` section below: its blast radius was too narrow when it named only `discussionparser`.
  Check that the readers it lists are complete.

## Merge danger

**Door:** two-way
**Blast radius:** records

The changed bullet text is read by `raddleparser`'s criterion and risk coverage, by `SplicePart`'s part records,
and by the acceptance coverage check of `CheckDecisionIDs`.
An in-flight raddle proposal that holds a cut criterion or risk now disagrees with its record,
and a binary on the old rule refuses a proposal filed on the new one.
Filing the proposal again with the binary that checks it clears either.
The change rewrites no stored record, so a revert undoes it.
