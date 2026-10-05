# lyx webster reset covers the report-head and last-batch-head targets

`webster-reset-targets` · Note · enhancement, webster

## Background

`webster-audit-recovery` (PR #381) added `lyx webster reset --to start|pre-fix` so recovery never needs a denied `git reset --hard`.
Its plan recorded one deviation not covered by the decision record (D7): three sites still tell the agent to run `git reset --keep <sha>` rather than a lyx verb.
They move back to the report head or the last-batch head in record-batch, recover-batch and accept-audit (`internal/websterengine/gitwrap.go`), plus standalone mode.
The verb was not widened because its targets are only `start` and `pre-fix`.
`--keep` is not denied and refuses rather than discards, so it was accepted at PR-Gate.

## Proposal

Widen `lyx webster reset` with the report-head and last-batch-head targets, and point the three sites and standalone mode at it, so every Webster recovery instruction names a lyx verb.
