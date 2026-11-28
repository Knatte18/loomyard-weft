---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Code review round 1, fixer report

All four findings are fixed as proposed.

## Changes

- F1 (`6df731725`): removed the first-caller rule from `loom-block-plan-cards.md`, `loom-rubric-plan-review.md` and `loom-plan-spec.md`, restoring those places to their base text.
  The card-claims and cards-against-each-other text stays.
  In `_lym/landing/summary.md` (edited, not committed): dropped the first-caller bullets and the intro sentence that repeated the false premise, and added that #290's ask is already met by `2326ef4da`.
  Premise checked: `cmd/lym/testonlycode_linux_test.go:91` carries `//lym:only upkeep`, `pattern/PATTERN-test-only-code.md` lets a card add a function only a later card calls, and `2326ef4da` is an ancestor of the base.
  Commit `e489adf46`'s message still states the old rule; it was not rewritten.
- F2 (`6947f06f8`): a fact a `fix` names as unverified now leads to the finding's ordered disposition, so Dispute applies.
- F3, F4 (`d654b4e6b`, then `0cd45a01f` from the self-check): `burler-review-format.md` now routes a disproved fact on a carried finding and on a kept parent finding.
- The summary's `## What changes` bullets are updated to match F2, F3 and F4.

Verification: `lym gate test ./contracts/... ./internal/burlerengine ./internal/planparser` passed (0 failed, 4 skipped), after the first three commits and again before `0cd45a01f`.

## Self-check

- fix-code-correctness: no findings.
- fix-affected-set: no findings.
- fix-consistency, LOW, the kept-parent bullet does not say it is an exception to the disproved-fact rule: fixed as proposed (`0cd45a01f`).
- fix-proposal, LOW, the same bullet names no actor and does not mark the exception: merged with the line above; fixed as proposed (`0cd45a01f`).

## Disputed

None.

## Deferred

None.

## Rulings

None.
