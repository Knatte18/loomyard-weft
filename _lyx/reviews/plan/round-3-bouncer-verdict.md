---
verdict: APPROVED
rationale: "Round 3's F1 is fixed. Card 8 now confirms an injection only by a turn end past the entry offset, a failing SessionIdle probe keeps waiting with the flag unchanged, a restarted watcher saves PhaseInjected false before its first check, and card 6 plus four new card 8 tests cover the clearing-timeout path. No other finding remains against D1-D16, verify coverage or granularity."
---

# Plan-Review round 3 — judgment

The review report has no `## Focus departures` section, so there is nothing to ratify.

## F1: fixed

The round-3 report's one finding, also the open ledger entry `failing-probe-confirms-injection`, is closed by the fixer's edits to cards 6 and 8.

- **Confirmation rule.**
  Card 8's landed check for `handoff-requested` and `resuming` now confirms only on "a turn end past the entry offset".
  A failing `SessionIdle` probe with no such turn end "means keep waiting with the flag unchanged, so a later tick checks again".
  It gives the reason: the probe also fails on dialogs, drafts and unframable captures.
  A passing probe with no turn end triggers the re-send.
  This matches D10's restart rule.
  A turn really in progress confirms through its own turn end, and an idle pane with no turn end re-types the stencil.
- **Clearing-timeout path.**
  After `clearing` times out with the probe still failing, a resume `Send` error leaves `PhaseInjected` false.
  Later ticks wait while the probe fails and re-send `PendingResume` once it passes, so the cleared session is no longer stranded.
  That was the harm D10's rationale names.
- **Restart.**
  A watcher that loads a non-idle phase for the same strand first saves `PhaseInjected` false.
  The check then runs on every tick until a turn end confirms the injection or a passing probe triggers the re-send.
  Card 6's `PhaseInjected` line records both the turn-end evidence and the restart reset.
- **Tests.**
  Card 8 adds the four cases the report asked for:
  the clearing-timeout-then-`Send`-error path, a `Send` error during a running turn, a handoff `Send` error behind a failing probe, and a restart in `resuming` with the flag persisted true.
  The earlier restart cases ("with the injection landed → no duplicate send", "not landed → exactly one re-send") still hold under the new rule.
  A landed injection produces a turn end past `PhaseEventsOffset`, and the restarted cursor starts at `PhaseEventsOffset`.

## Everything else

Every Decision D1–D16 and every Constraint is still carried by some card, and no card adds scope the decision record does not license.
The `## verify:` section runs every targeted package, both untagged and under `-tags integration`, and only vets `smoke`, which the rubric allows.
Card granularity and `ImpactSummary` content are unchanged since round 2, where neither raised a finding.

One residual race was considered and not raised.
A turn could end with its Stop event not yet read when the probe passes, which would cause one duplicate re-send.
That behaviour is D10's own rule ("an idle pane with no such turn end means it did not, so re-type"), and it only arises after a `Send` error or a restart.
A duplicate handoff instruction for the same path, or a duplicate resume prompt, is harmless under D10's rationale.
