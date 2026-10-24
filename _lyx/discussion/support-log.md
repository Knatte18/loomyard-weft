# Support log — record-batch accepts a Create target declared in the unit's external test package

## Interview

A darn run holds no interview; the writer read the board entry and the code and made its own calls.

## Rejected alternatives

- A plan-time validator rejecting or rewriting `<dir>#Member` to `<dir>_test#Member`: at plan time a new external test file does not exist yet, so the validator cannot tell which package the member will land in.
- Resolving every Create target's counterpart up front in the batched resolve: doubles the resolve for every target, while the declarations listing already loaded for a missing member answers the question.
- Restricting the counterpart to `Test*` names: helpers and fakes in an external test package hit the same mismatch.

## Review rounds

_No rounds yet._

## Question ledger

- Fix at record time or plan time? Self-pick: record time, per the first rejected alternative.
- One direction or both? Self-pick: both, since the mismatch is symmetric.
- Does handle binding need the same fix? Self-pick: yes; it matches delta symbols by the expected glyph and would raise `bind-count-mismatch` once the done-check passes.
