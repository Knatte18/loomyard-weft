# Round 2 fixer report — discussion

All six findings validated against the code and fixed.

## Fixed

- **F1** (rubric backstop): D6 adds a second Plan-Review rubric finding, an `Edit` member glyph without the re-sign arrow while the Intent changes its parameters, results, type parameters or receiver.
  Open risks now names that finding as the only backstop and its limit; the stencil/rubric criterion names both findings.
  Verified: `contracts/stencils/loom/loom-rubric-plan-review.md` has no signature or caller finding today.
- **F2** (wrong transition): verified that `CanonicalizeHandles` runs inside `resolvePass` at every gate and rewrites to `plan:<glyph>`, which `IsHandleRef` still reports as a handle, and that `BindHandles` at record-batch turns it into a glyph.
  D3 now says so, counts a `Rename` to-side handle under its Old glyph's unit (what canonicalization writes), and its bound names record-batch binding.
  The `rework-generations` criterion and the handle redundancy criterion are restated around binding.
- **F3** (arrow placement): D5 states `planparser` parses the arrow, and the head-to-member match calls `quarry.Name` (pure, no I/O, `planparser` does not import quarry) in `planglyph` inside `resolvePass`, so it runs at every gate including dispatch, reading only plan text.
  An arrow on a `plan:` handle is a blocking finding; the arrow criterion includes self glyph and handle targets.
- **F4** (Goal over-claim): the Goal's verdict-stability sentence is scoped to checks this task adds or changes.
- **F5** (stencil sentence owner): D4 removes the check and spec row only and leaves the stencil sentence to D6.
- **F6** (restatements): removed D1's failed-card recovery sentence (Q13 holds it); moved the `webster-parallel-execution` paragraph to the support log's Interview item 2; cut D7's second sentence to the flip assignments, now naming the D4 flip; dropped the per-card green-suite criterion, which D1 states.

Support log: Interview item 3 wording follows F2; ledger gains Q16–Q19 for this round's picks.

## Disputed

None.

## Deferred

None.
