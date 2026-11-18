---
round: 2
ledger:
  - key: card3-drift-testenv-seams-unlisted
    rounds: [1]
    status: resolved
  - key: card10-routing-progressat-pin-unlisted
    rounds: [1]
    status: resolved
  - key: card7-review-input-note-fixed-at-build
    rounds: [1]
    status: resolved
  - key: card8-findings-file-location-dirty-tree
    rounds: [1]
    status: resolved
  - key: card7-shed-recipe-spec-reviewinput
    rounds: [1]
    status: resolved
  - key: card10-prreview-stuck-always-halt-docs
    rounds: [1]
    status: resolved
  - key: card9-prreview-type-doc-unlisted
    rounds: [1]
    status: resolved
  - key: card3-trailer-escape-hatch-unbounded
    rounds: [1]
    status: resolved
  - key: card3-card9-battencli-intentionallynil-unlisted
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card7-capturereview-signature-breaks-tests
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
---

Round 1's eight findings are all resolved per round 2's review.
Round 2 raised two new BLOCKING scope gaps, both tests a card breaks without listing them.
Card 3 and card 9 add required seams that batten leaves nil, so `intentionallyNil` in `internal/battencli/wire_seams_test.go` needs entries.
Card 7 changes `captureReview`'s signature, which stops two table-driven tests compiling.
