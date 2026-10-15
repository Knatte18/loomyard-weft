# Fixer-report, webster round 1

## Changes

- F1 (fixed as proposed): `PATTERN.md` carries card 14's entry text for `PATTERN-shuttle-provider-seam`, naming both the containment half and the import half.
- Gate finding (overview past the headroom line): the F1 entry pushed `PATTERN.md` 70 bytes past the headroom line.
  The `PATTERN-cwd-resolution`, `PATTERN-hub-containment`, `PATTERN-glyph-conversion-chokepoint` and `PATTERN-edit-directive` entries are shortened, and each entry's detail is already in its background file.
  `PATTERN.md` is now 15562 bytes, under the 15564-byte headroom line.
  `lyx gate test ./internal/pattern ./cmd/lyx` passes.

## Disputed

None.

## Deferred

None.
