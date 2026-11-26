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
or when it is unindented and opens no heading, HTML block or blockquote, as markdown reads a lazy continuation.
A blank line, the next bullet, a fenced block or a carry-over block still ends the bullet.
`SplicePart` already writes a carried bullet's continuation lines indented, so a spliced part record reads back the same text.

`contracts/specs/loom-discussion-spec.md` now states where a bullet ends.

## Design decisions

- The parser reads the line as markdown does, rather than the record format check refusing it.
  A refusal would stop a record a human writes in ordinary markdown, and the stored record that hit this would still need a rewrite;
  the parser change makes every such record read as it renders.
- The HTML-block exception keeps a carry-over block's `<!-- lym:carry-over … -->` marker, or any other comment, from joining the bullet above it.

## Tests

- `TestSectionBullets` gains `UnindentedContinuationLine`, the reported case, and `HeadingEndsBullet`.
- `TestCheckDecisionIDs` gains `IDOnUnindentedContinuationLineIsCovered`, a decision id named only on an unindented continuation line.

`UnindentedContinuationLine` and `IDOnUnindentedContinuationLineIsCovered` fail on the merge base;
`HeadingEndsBullet` passes there and pins that a heading still ends a bullet.

## Merge danger

**Door:** two-way
**Blast radius:** discussionparser
