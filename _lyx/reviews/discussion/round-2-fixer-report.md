# Fixer report — discussion round 2

All four findings were validated against the code and fixed.

## Fixed

- F1 (MEDIUM, design): D6 now follows `RecoveryRetry` alone for a dead recovery, matching Merriam's ladder (`internal/websterengine/doc.go:383`).
  An unrendered amendment only skips the exhausted case, because recover-batch admits that case (`recoverbatch.go:440`).
  The acceptance row for a dead recovery at the cap with an unrendered amendment now expects "end stuck" and never "exhausted".
  The rejected alternative, letting an amendment earn a retry at any count, is in the support log.
- F2 (MEDIUM, design): D2 treats an unquoted leading `^` as a glob character, under the same rule as `*`, `?` and `[`.
  The bound now names any glob option and gives the quoted and range forms for a revision exclusion.
  `plainWord` (`refscanner.go:263`) admits `^` today, so the premise holds.
  New rows: admitted `git log '^main' feat` and rejected `git log ^main`.
- F3 (LOW, scope): D6's exhausted case now covers every terminal status except done, so the stuck and failed branch no longer names a recover-batch that would refuse at the cap.
  The spec-row rewording covers the five cases, and a new row covers a stuck record at the cap with no unrendered amendment.
- F4 (LOW, consistency): added the row for a dead fork record at the cap with an unrendered amendment, which names `recover-batch NN` and never "exhausted".

The support log's interview item 7, the question ledger and the rejected alternatives were updated to match.

## Disputed

None.

## Deferred

None.
