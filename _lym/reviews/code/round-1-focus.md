---
round: 1
focus:
  - "internal/shuttleengine/startup_test.go, the shared `elapsed >= wantHeld + apiErrorRetryBackoff` bound in TestStartup_LoadTurnAPIError: for every succeeding row, is the bound sound against the row's real virtual time (poll intervals, busy-session back-offs, the hangs and dies rows), and does it still distinguish a tripled or ceiling-overshooting back-off from the correct one, or can a row's wantHeld be so loose the bound passes anyway?"
  - "The removed `wantHeldAtMost` on the ceiling row: does the ceiling row's wantHeld, `apiErrorRetryBackoff*(1+2+4+8) + 2*loadHoldBackoffCeiling`, equal the schedule `apiErrorRead` and `loadSkillTurn` in loadturn.go produce, so the new upper bound pins the ceiling exactly and not just from below?"
  - "New row 'an open turn past its timeout that then ends loads its skills': does `lateEnds: {0: 8}` with `SkillLoadTimeout: 50ms` really hold the turn open past the timeout in the fixture, and does the row fail if loadSkillTurn returned nil/ended=false on the timeout (skills skipped, prompt typed) instead of awaiting the open turn? Check it against the existing open-turn rows for duplicated coverage."
  - "New row 'an open turn after a hold fails the start at the deadline the hold moved': does `wantFailedAfter: time.Hour + apiErrorRetryBackoff` follow from `heldDeadline` and `endLoadHold` in loadturn.go, and does the fail path check an upper bound, so a deadline moved by too much would still fail the row? Compare with the existing deadline row for overlap."
  - "Scope against the decision record's two open checks (`hold-behaviors-lack-pinning-rows`, `open-load-turn-skips-and-types-prompt`): which listed hold behaviors does the diff leave without a pinning row, and does the summary's claim that the others were already pinned hold against the table's rows? The diff is test-only; check the summary's 'no production code changes' claim."
  - "Test economy of the diff: any new row or assertion that repeats a covering row of TestStartup_LoadTurnAPIError or of the other load-turn tests in startup_test.go."
---

Seed pass only; the diff is one test file plus the change description, so the focus is the test's soundness against `loadturn.go`.
