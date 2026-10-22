# Support log — twill-plan runs 5 lenses instead of 8

## Interview

A darn run holds no interview: the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- Keep the four unused lens definitions in the template: rejected, since no fan would run them and the board entry keeps only definitions another fan still uses.
- Add `forced-edits` to `review-plan` or `affected-set` to `twill-plan` to satisfy the perch-parity test: rejected, since `review-plan` is out of scope and the operator's table drops `affected-set` from `twill-plan`.
- Drop the perch-parity test: rejected, since it still guards every other reviewer lens.
- Treat the twill-plan lenses as procedure lenses exempt from the emphasis line: rejected, since the exemption is defined by the review and fixer fans and widening it changes a rule beyond this task.

## Review rounds

_No rounds yet._

## Question ledger

- Do the deleted lenses stay defined? Self-picked: no, per the board entry's rule that only a lens another fan uses stays (D3).
- How does the perch-parity test treat `affected-set`? Self-picked: an absorbing-lens mapping for that one pair (D4).
- Does a language layer need a change? Resolved: no layer defines or orders these lenses.
