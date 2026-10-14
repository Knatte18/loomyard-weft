## Changes

- F1: fixed as proposed in spirit, with a different mechanism than the review's example.
  `LaterDeleteReferences` (`internal/planglyph/deleteorder.go`) builds `allRegions` from later plus deleting cards, then filters to the cards in `later` for the reported `regions`.
  `deleteOrderFindings` and `unrewrittenLaterEditRegions` take both sets: holders come from `regions`, the rewritten-line exemption consults `allRegions`.
  `laterImportFindings` and `regionFiles` use only `regions`, so a same-batch higher-numbered deleting card is never reported.
  The caller-coverage call in `callercoverage.go` passes its single region set twice.
  Regression row added to `TestLaterDeleteReferences` (new `siblings` field): a higher-numbered same-batch card whose Edit span references the deleted member yields no finding.
  The mechanism filters by card number of `later` at region level instead of recording card numbers for `deleteOrderFindings` to check; the behavior is the one the finding asks for.

`lyx gate test ./internal/planglyph` is green.

## Disputed

None.

## Deferred

None.
