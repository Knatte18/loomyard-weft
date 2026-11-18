# List features nobody uses, for the operator to cut

`drop-unused-features` · Task · enhancement, tools · starting · Open-Window

## What was seen

The `landingshed` reading found about 340 lines of features a product owner could remove
(the flaky rerun, the adopter-first repair note, the beam landing record among them); nobody has checked whether they are used.
Code that no path reaches was removed by `remove-unreachable-code`, which has landed; this note covers features that are reachable but unused.

## Idea

A list, not a deletion: features, flags, config keys and verbs with no use in run records, transcripts or hub configs over the last weeks.
The operator decides which go; only those are removed.
