# Fixer report — discussion, round 1

## Changed

- F1 (BLOCKING): D4's pack-on FRESH-READ now has the fork, in the turn right after the card-start read and before any edit, read whole every target and `Uses:` file no listed span covers (a plain path, a self glyph's file, an unresolved member glyph once located).
  D3's rationale now says the card file and its listed spans are one turn, with files only the card file names following in the next.
  Added the Acceptance criterion that the pack-on FRESH-READ text has the fork read those files whole before any edit.
  D4's Bound now speaks of target and `Uses:` files that hold a listed span, so it no longer contradicts the new uncovered-file read.
- F2 (MEDIUM): D3's pack-on card-read fragment now reads a file an earlier card of the batch edited whole in the same turn and finds the member by its listed signature, instead of reading its spans by offset.
  D4 says that whole-file read is the fresh read of such a file.
  Open risks now carries only the residual one-line `var`/`const` signature risk.
  The card-read Acceptance criterion names the whole-file read too, so the carve-out is tested.
- F3 (LOW): D3 states the pack-on fragment keeps step 1's two following sentences verbatim, and the card-read Acceptance criterion names them.
- F4 (LOW): D1 and the quarry-unavailable Acceptance criterion now list member glyphs with the error and `plan:`-handle refs with the handle reason.
- F5 (LOW): D2 renders `none` for the entries of a card whose member glyphs all failed to resolve, followed by its unresolved list; the per-card Acceptance criterion covers that case.
- F6 (LOW): support-log Interview step 3 now says a grouped spec's signature is its keyword prepended to the spec's text, so the first line reads `var name …` or `const name …` in both forms.
- F7 (LOW): Scope Out's measurement bullet is cut to its one line.
  The removed sentences moved to support-log Interview step 5 under the board-fit MEDIUM entry.
  The goal-scope LOW entry that said "fixed, Scope Out says queuing it is the operator's call" now reads "settled", since Scope Out no longer carries that line.

## Disputed

None.

## Deferred

None.
