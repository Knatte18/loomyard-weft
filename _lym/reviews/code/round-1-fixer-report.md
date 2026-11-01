# Fixer-report, round 1

## Changes

- F1 (LOW), fixed differently. Commits `a7655295` and a follow-up.
  Proposed: reword the comment above `Notice` in `refuseRawRow` to "The notice names only what is refused and the scopes it adds; the gate rule itself is stated by another row's notice, see the package doc."
  Not made as proposed: the "another row's notice, see the package doc" clause is a cross-symbol dependency and a "see also" pointer, which `code-quality`'s Comments rules prohibit (fix-proposal lens).
  Made: the comment reads "The notice names only what is refused and the scopes it adds."; the cross-row relationship stays in the `doc.go` paragraph.
- F2 (NIT), fixed as proposed. Commit `7f24080d`: `doc.go` now says "while the steers of the raw go and refuse-raw rows each carry the full rule".

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: LOW on F1's proposed wording; fixed with my own change (above).
- fix-proposal: F2, no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
