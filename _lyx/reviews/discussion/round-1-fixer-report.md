# Discussion-Review round 1 — fixer report

All six findings are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` is unchanged: it already records the hub-review provenance removed from the decision record.
`lyx loom validate-discussion` exits 0 after the edits.

## Changed

- **F1 (MEDIUM).**
  Decision 1's Bound now says subagents and forks run in their parent's permission mode, so under `bypass` every agent orch spawns runs with no prompt, and that an adopted session comes back under orch's mode.
  Decision 3's Bound now says the allowance has no operator switch, and that `permission_mode: prompt` is the remaining lever.
- **F2 (MEDIUM).**
  Decision 5's Bound is narrowed to a "positively identified" live holder.
  It now states that an absent or unreadable registry lets an unconfirmed holder through, bounded by the explicit operator verb, the warning (now also on the envelope), and the prime-only transcript check.
  The adopt-preflight acceptance line now requires the envelope warning.
- **F3 (MEDIUM).**
  Decision 6 now defines the DEFER match: the turn end's final assistant message, trimmed, equals `DEFER` exactly.
  It settles the open cases: a written handoff file wins over `DEFER`, a turn end with neither keeps today's wait-then-timeout, and `DEFER` on a hard or requested cycle is ignored.
  The deferral holds until `soft_idle_s` after the DEFER turn end was read.
  The watcher acceptance line covers each case.
- **F4 (MEDIUM).**
  Decision 6's soft condition requires the reading to be below `threshold_tokens`.
  A hard-cap reading is always a hard trigger whatever the idle knobs are set to, so a soft threshold at or above the hard one never fires.
  The watcher acceptance line now covers `idle_grace_s` above `soft_idle_s`.
- **F5 (LOW).**
  Decision 7 now includes the adopted strand's name, parentage and SendMessage reachability in its smoke scope, and says this verifies shipped agent-naming with no new code.
  The acceptance line is labelled as a live-smoke check.
- **F6 (NIT).**
  The "(hub review, 2026-10-02)" and "(revised by the hub review)" parentheticals are gone.
  The first auto-mode bullet now credits Decision 6's natural-break rule to the hub review.

## Deferred

None.
