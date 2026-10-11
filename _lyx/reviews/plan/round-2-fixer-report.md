# Plan-Review round 2 — fixer report

## Changes

- F1 (LOW, consistency), fixed as proposed (first alternative): `_lyx/plan/01-claude-lone-eol-mark.md` now has the fake reed serve `pane-idle-eol-mark.txt` on every capture read, replacing the three-phase sequence.
  Premise checked against `internal/shuttleengine/send.go`: `confirmPasted` returns nil when the box reads empty (`case box == ""`), and Claude's `PasteSubmits()` is true (`pasteSubmitsOnEnter`), so under card 1's own reading a lone-`%` capture lands each paste on its first confirm read.
  The middle state the old text left open is gone, and each send's pre-write guard still reads the lone-`%` box.

## Disputed

None.

## Deferred

None.
