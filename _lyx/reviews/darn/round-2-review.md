---
verdict: APPROVED
---

No findings.

The three focus questions are settled as follows.

- A card present only in `deleting` is never a holder: `LaterDeleteReferences` filters `allRegions` down to cards in `later` before building `regions`, and only `regions` feeds the reported holders in `unrewrittenLaterEditRegions`.
  The same card still serves in `allRegions` as an exemption for references its own Edit span rewrites, when its number is at or below the deleting card's.
- Webster's `recordbatch.go` passes the batch's own cards as `deleting` and `unbegunCards` as `later`.
  A higher-numbered same-batch card that is not in `later` is neither a holder nor an exemption for a lower-numbered deleter, so no finding names a card outside the unbegun later cards.
- The `TestLaterDeleteReferences` row "a same-batch deleting card numbered higher is not reported as a later holder" pins that case: card 2 edits `sub#Bar`, which holds `Foo()`, and card 1 deletes `sub#Foo`.

The diff matches the amended board entry: exemption for the deleting card and earlier cards, rows for each case in the existing table, and the `doc.go` entry.
