# Round 5 fixer report

All six findings fixed in `_lyx/discussion/decision-record.md`; `_lyx/discussion/support-log.md` gained Q18–Q21 in its Question ledger.

## Changed

- F1: D6's revive now reaches a dead driver strand only, and says a retiring strand is left to `lyx loom start`.
  D7's no-live-driver outcome splits into dead, retiring and absent strands; the retiring refusal names `lyx loom start` only.
  The `lyx loom resume` Acceptance row and the `batten run` row match.
- F2: D7's awaiting refusal names approve or reject, and `lyx batten run <slug>` when no batten run watches, and reads no batten state.
  D6 keeps the acted-on hint's `lyx loom start` as its driver-ended fallback and says no awaiting hint names `lyx loom resume`.
  The `batten run` Acceptance row says the same.
- F3: D7's rationale now says `lyx loom start` wakes a parked live driver and spawns only when none is live.
- F4: every `lyx loom start --no-attach` in D6, D7 and the Acceptance rows is now `lyx loom start`; the halt-reason Acceptance wording no longer relies on "alone".
- F5: the Goal, Scope, D3, Constraints, Auto-mode assumptions and Acceptance rows use "timeout".
  D3 makes it a `time.Duration` parameter of `verifytree.Verify` with the constant `verifytree.Timeout`, and says `Verify` derives its own context.
  "Bound" stays for D3's `WaitDelay` sentence only, now worded "limit".
- F6: "the child's run lock" for the loom run's lock in D6, D7 and the Acceptance rows, and "batten's own run lock" in D7's parenthetical.

## Deferred

None.
