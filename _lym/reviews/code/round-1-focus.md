---
round: 1
focus:
  - "internal/heddleengine/shipped.json: does the new `lym-plan-edits-before-gate` entry sit after `lym-batching`, target `claude-file`, and word the instruction as a command covering all four steps (one-message reads with whole files or every span and a second round only for files the first round named, every edit decided before the first, all edits in one message, one gate run with its fixes in one message)?"
  - "Same entry: does it name every forbidden pattern the record lists (one file read per turn, one file read in slices over turns, edit-read-edit over turns, a gate run between edits of one change)?"
  - "Stencil diffs under contracts/stencils: is every batching line `turn-batching` added removed, with no rule the record did not ask to change dropped or weakened, and the card-specific parts kept (spans a Webster card names, a darn commit as the unit of one change)?"
  - "Stencil diffs: does any removal go beyond batching lines, or does a batching line remain in a stencil the diff did not touch (grep contracts/stencils for leftover batching wording)?"
  - "internal/heddleengine/store_test.go: does the test change cover the new entry's behavior without duplicating an existing test, and are the shipped layer's other tests still consistent with the added entry?"
  - "Commit message and diff versus the record: does the diff stay within what the record licenses (no heddle-side changes, no change to the appended system prompt), per `PATTERN-lym-instructions-in-code`?"
---

Seed focus from the decision record's Goal and the single commit in the range.
