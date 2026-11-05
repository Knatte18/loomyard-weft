---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: internal/creelengine/deliver_test.go:2406
    summary: no test row pins the zero TurnEndedAt branch of woken, the case the new doc.go Bound line documents
    fix: In TestScheduler_HookWake in internal/creelengine/deliver_test.go, add one row after the two new rows. It calls e.del.SetTurn(alpha, false, time.Time{}) before ackedRing(t, e, "one"), posts "two" with settle, and asserts untilBell(t, e, ringAt) returns at least 30*time.Second, since a reading with no turn end time holds the bell for the window.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/creelengine/deliver_test.go:1710
    summary: the TestScheduler_HookWake doc comment names two bell cases and omits the new hold while no turn end follows an answered ring
    fix: In the doc comment above TestScheduler_HookWake in internal/creelengine/deliver_test.go, extend the third line so it also says the bell is held within 30 seconds of a ring while no turn end was read after it, since the woken turn's Stop hands out the mail.
    origin: lens:code-affected-set
---

### [LOW] No row pins the zero TurnEndedAt branch of the hold

**Location:** internal/creelengine/deliver_test.go:2406
**Issue:** Both new rows set a non-zero `TurnEndedAt` (a minute before the ring, a second after it).
`woken` also turns true for a zero `TurnEndedAt` after a ring: a pane-decided reading, or an interrupted woken turn.
`doc.go` records that case as a Bound line, and no row exercises it.
A change to `idleness.TurnEndedAt.Before(mark.at)` would pass every row.

### [LOW] Test doc comment omits the new hold

**Location:** internal/creelengine/deliver_test.go:1710
**Issue:** The comment says the bell follows "after the grace of 5 seconds or after a ring unanswered for 30".
The diff adds a third case the new rows pin: within the window the bell is also held while no turn end was read after the ring.

## Rejected

- Lens code-correctness returned no findings; its branch list (zero `mark.at`, burst case, zero or equal `TurnEndedAt`, past-window answered and unanswered rings, `belled` and `Idle` error paths) matched my own reading of `deliver.go:259-295`.
- Whether the diff reaches the goal's quoted reminder line: the line quoted in the Goal is the text of `BellLine`, which `doc.go` quotes verbatim as the hook strand's bell, so the hook-strand bell is the path the Goal describes and the diff's scope fits it. Not a defect.
- `summary.md` claims (hold within the window, grace otherwise, two test rows, existing rows unchanged) match the diff; the 17 s and 8 s figures quote the Goal. No finding.
- Commit messages carry no `Pattern-Departure:` trailer and no claim that departs from the Goal. No finding.
