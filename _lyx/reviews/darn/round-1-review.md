---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelengine/journal_test.go:438
    summary: no test carries a hold whose last record is renewed across a day rollover, so the origin set by the renewed fold case is unpinned
    fix: In the existing retention carry test near journal_test.go:438, add a renewed record with a later expiry after the held record for the same receiver, and assert the hold carried to the next day keeps the renewed expiry and owner. Extend that test; add no new test function.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelengine/verbs_test.go:538
    summary: nothing pins that a hold taken again after the earlier one expired writes a held event, which the new event choice and the decision record's second decision rely on
    fix: Extend the Hold_OwnerRenewsAndReleases row, or the existing expiry row near deliver_test.go:487, with a second Hold by the same owner after the fake clock passes the first expiry, and assert the log reads held, held. Add no new test function.
---

### [LOW] Renewed hold carried across a day rollover is untested

**Location:** internal/creelengine/journal_test.go:438
**Issue:** The `eventRenewed` fold case in `fold.go` sets `origin: day`, which `carryRecords` reads to decide what to carry.
The renewal row stays on one day and the carry fixtures build holds from `held` records only, so dropping or breaking that `origin` would pass every test.
The code itself matches the `eventHeld` and `eventCarried` paths; the gap is coverage only.

### [LOW] Hold taken again after expiry is untested

**Location:** internal/creelengine/verbs_test.go:538
**Issue:** `Store.Hold` writes `held` rather than `renewed` for a hold that expired unreleased, because `liveHoldLocked` treats an expired hold as absent (`pool.go:208`).
The code is correct, but no test pins the event written in that case, and the decision record names it as a decision.

## Rejected

- Older binaries keeping the hold's first expiry: the decision record's declared open risk and consistent with `PATTERN-tolerant-records`, not a defect.
- Journal growth of one line per renewal: an accepted assumption in the decision record and within the board entry's scope.
- `renewed` missing from other journal readers: no other reader of event names exists beyond `fold.go` and `verbs.go`; the only other place naming `held` is `doc.go`, which the diff updates.
- Summary claims (restart behavior, older-binary tolerance, test coverage): each is borne out by the diff and the changed test row.
