# A decision record criterion with an unindented continuation line is cut at its first line

`record-criterion-line-cut` · Task · bug, parsers, loom · starting · Seed-Child

## Seen (raddle of `sandbox-closed-area-build`, 2026-11-26)

The stored record's D6 attach criterion runs over two lines, the second not indented under the bullet.
The record parser keeps only the first line as the criterion.
Raddle could not carry the whole bullet: joined to one line, the raddle-propose gate refused it as criterion-unknown, since it matches only the cut text.
The second clause survived only by being restated in the part's contract.

## Idea

Either the record format check refuses an unindented continuation line when the record is stored,
or the parser reads it as part of the bullet, as markdown does.
