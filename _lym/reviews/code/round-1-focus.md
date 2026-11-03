---
round: 1
focus:
  - "internal/shuttleengine/wait.go pollEventsTick gated branch: with the waiting turn end kept (countedTasks/waitingTasks no longer cleared) and no Done returned, does the run reach its next turn end correctly when the shell completes, and can a stale waitingTurnEnd or waitingTasks leak into a later tick, a hold, or the plain-turn-end path?"
  - "internal/shuttleengine/wait.go expiredTurnEnd gated branch: the guard `!filesExist || !unawaitedShellsPastBound` returns keep-waiting; does a gated waiting turn end with a missing output file still hold at once as the doc claims, and are the ungated paths (Done at once with files present, hold after the bound with a file missing) unchanged after the refactor into unawaitedShellsPastBound?"
  - "internal/shuttleengine/wait.go shellFirstSeen bookkeeping: is every shell's first-seen time recorded before expiredTurnEnd reads it in the gated path (a zero time would make the bound pass at once), and is the bound measured from first sighting rather than from the turn end?"
  - "internal/shuttleengine/wait.go queueShellNudge / sendShellNudge: once-per-shell naming across turn ends, the `turnStartAfter` guard that skips queueing, the pause (`run.covered`) drop, and the failed-send path; does dropping under a pause or a failed send mark the shell as named so the agent is never told, and is sendWithin safe to call at that point in the Wait loop (deadline, locking, interaction with settleDeferredHold)?"
  - "Scope against the Goal: the board entry offers refuse/downgrade `done` or stop shells at run end, and asks to check a gated run abandoned this way releases its slot. Does the diff make a change the Goal licenses, and is the gate-slot claim in _lym/landing/summary.md (OS file locks, TestAcquire_HolderProcessDeathFreesSlot) borne out by internal/gateslot?"
  - "internal/shuttleengine/waiting_test.go: do the changed and new tests cover the nudge, the bound, and the staggered two-shell case without duplicating an existing test; is any added behavior (pause drop, failed send, turnStartAfter skip) left uncovered?"
  - "Doc comments in config.go, template.yaml, shuttleengine/doc.go, wait.go and websterengine/doc.go: do they agree with each other and with the code on the gated expiry to Done, and follow the repository's comment conventions (line breaks and wrapping excluded)?"
---

Seed focus for round 1. The record holds the board entry alone, so the entry is the measuring stick and `_lym/landing/summary.md` is a claim to check against the diff.
