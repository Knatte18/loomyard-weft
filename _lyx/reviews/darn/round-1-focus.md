---
round: 1
focus:
  - "internal/creelengine/fold.go `apply`, `eventRenewed` case: does the hold it builds match what `eventHeld` and the day-rollover carry (`eventCarried` with `Of: eventHeld`, around fold.go:333 and :445) build, so a hold renewed on one day and carried to the next keeps its renewed expiry and owner?"
  - "internal/creelengine/verbs.go `Store.Hold`: is `Renewed: current != nil` the right condition for writing `renewed`, given a hold that is expired but not yet released (does `current` treat an expired hold as live, and does the log then show a start for it)?"
  - "Journal consumers beyond the fold: does any other reader of journal records (pruning, compaction, replay on restart, `creel log` filters, doc listings of event names) need to know `renewed`, or fall into the unknown-event Debug path?"
  - "The board entry asks for one held event per hold start and none per refresh: does the diff cover it and the entry's 'one when it ends or changes' wording without adding scope, and does `_lyx/landing/summary.md` claim anything the diff does not bear out (restart behavior, older-binary tolerance, test coverage)?"
  - "Test economy: does the changed `Hold_OwnerRenewsAndReleases` cover the fold's `renewed` branch and the restart persistence, and does it duplicate an existing test?"
---

Small diff in `internal/creelengine`; the risk is in the fold and carry paths rather than the writer.
