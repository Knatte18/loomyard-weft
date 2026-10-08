# Fixer report — discussion round 4

Every finding was validated against the code and fixed.

## Fixed

- **F1** (MEDIUM, design): D5 now states coverage direction per kind: a `Delete` caller by the same or an earlier card, a `Rename` Old side's or re-signed member's caller by the same card only, with the reason.
  An acceptance criterion pins the difference; Q6 is refined and Q21 records the pick; the earlier-card option is a rejected alternative.
- **F2** (MEDIUM, design): confirmed at `internal/planglyph/deleteorder.go:203` (`\bName\b` per line).
  D5 now detects references with a `go/scanner` token walk that skips comments and strings, and matches a package-level name only as an identifier not preceded by `.` in its own package; `deleteorder.go`'s patterns are not reused.
  The over-reporting Bound names what still matches (local identifier, struct-literal key, same-named method) and that covering the file or package clears it; the Open risks line now points at that bound.
  An acceptance criterion pins the comment, string and selector cases; Q22 and a rejected alternative record the pick.
- **F3** (LOW, consistency): confirmed `LaterDeleteReferences` is called inside `resolvePass` (`internal/planglyph/planglyph.go:352`).
  D5 drops "beside `delete-before-reference`", and the plan-writer note says the call site is in `resolvePass`, shared with dispatch, and caller coverage goes in the plan-gate-only pass.
- **F4** (LOW, consistency): Scope now carries D6's qualifier, "only Go file paths of non-test files".
- **F5** (LOW, scope): new acceptance criterion: the same member re-signed on two cards validates clean, with no `handle-canonical-collision`, and no arrow rewrites the plan on disk.
- **F6** (LOW, scope): the Rename to-side criterion now names the discriminating case: a self glyph of the draft handle's package with the Old glyph in another unit gets no finding at either gate.
- **F7** (NIT, consistency): D5 says "under the unit of its glyph" and no longer names `Glyph.UnitPath`.

## Disputed

None.

## Deferred

None.
