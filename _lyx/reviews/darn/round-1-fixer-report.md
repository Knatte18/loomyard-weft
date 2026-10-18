# Fixer-report, round 1

## Changes

- F1, fixed as proposed and extended: `TestJournal_Retention` (`internal/creelengine/journal_test.go`) gains a `renewed` record after the day-one `held` on `beta`, asserting the carried hold keeps the renewed expiry and owner.
  The self-check showed that alone does not pin `origin`, because an empty `origin` also sorts below the cutoff day and is carried.
  A second hold, on `alpha`, is held on day one and renewed on day five; the test asserts it survives with the renewed expiry and is not carried.
  With `origin: day` removed from the `eventRenewed` fold case, the test fails; it passes with the case restored.
- F2, fixed as proposed: the `Hold_OwnerRenewsAndReleases` row in `internal/creelengine/verbs_test.go` gains a second Hold after release and a third after the fake clock passes its expiry, asserting a fresh hold and the log `held, released, held, held`.
  The row is renamed `Hold_OwnerRenewsReleasesAndRetakesAfterExpiry` to match.
- The day-one comment in the retention test names the renewal.

`lyx gate test ./internal/creelengine` is green.

## Self-check

- fix-consistency, LOW: the day-one comment omitted the renewal; fixed.
- fix-consistency, LOW: the row name covered only renew and release; fixed by the rename.
- fix-code-correctness, LOW: the renewal test did not pin `origin`; fixed with the day-five renewal on `alpha`.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
