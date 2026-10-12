# Fixer report — discussion, round 2

## Changes

- F1 (MEDIUM, consistency): fixed as proposed.
  D4's pack-on sentence lists only a plain path and a self glyph's file for the turn right after the card read, and a new sentence has an unresolved member glyph's file read whole in the turn right after the `lyx quarry resolve` call or grep that locates it, still before any edit.
- F2 (LOW, design): fixed as proposed.
  D3's card-read paragraph and its locate-fragment description, and the card-read acceptance criterion, now find a member in a file an earlier card edited by its glyph's member name, with the listed signature as a cross-check.
  Open risks reads "- None."
  The same change also renames D4's Bound reference from "D3's signature rule" to "D3's member-name rule", so it agrees with the reworded D3.

## Disputed

None.

## Deferred

None.
