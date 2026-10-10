# Go accepts a CONVERGED verdict with an open BLOCKING finding

`judge-converged-open-blocking` · Note · bug, burler, loom

## Seen

The haiku-roles analysis (`docs/research/haiku-judge.md`, 5459376bb) found that Go accepts a CONVERGED verdict even when a BLOCKING finding is open, and that the findings carried into the decision record come only from the judge's own ledger.

## Idea

Refuse CONVERGED in Go while any BLOCKING finding is open, and check that every finding key from the previous ledger is still present, so a weaker judge model cannot drop one silently.
