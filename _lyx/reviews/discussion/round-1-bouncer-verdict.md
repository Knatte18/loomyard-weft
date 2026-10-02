---
verdict: APPROVED
rationale: "All ten round-1 findings are legitimate and fixed in the current decision record: Decision 5 makes the store count decide the cap via a terminal GateResult, keys the Pending/Final lift on a wiring-only GateEntry field with Final failing on an unconsumed reject, defines fresh-spawn round retention and the three-condition halted-at-cap test, Decision 4 bounds failed liveness reads and the nested-pair timed pass, the sixth constraint is narrowed to the orch session, and no new unbounded edge remains"
---

# Discussion-Review, round 1 — judge verdict

The report carries no `## Focus departures` section, so there is nothing to ratify or reject on that front.

## Findings ratified

Each finding was checked against the rubric's `Do not flag` list and the writer's stencil.
None asks for a Notes subsection, rejected alternatives in the decision record, or call-site enumeration, so all ten stand as legitimate.

- **F1 (BLOCKING), two cap counters.** Legitimate: `handleGatedBoundary` fails only once the in-memory count reaches `Attempts`, and that count resets on attach.
  Fixed: Decision 5 now says the round store decides the cap, `attempts: 3` is told to the gate, the cap's reject returns a new terminal failure on `GateResult`, and the in-memory count never exceeds the store count.
- **F2 (BLOCKING), unkeyed Pending/Final lift.** Legitimate under the attack-surface rule.
  Fixed: the lift is keyed on a new `GateEntry` field only the parent-review wiring sets; the Bound lists exactly what passes; `Final` fails on an unconsumed reject, recorded as an auto-mode pick and a rejected alternative.
- **F3 (BLOCKING), fresh-spawn `BeginRound`.** Legitimate: without it the way out and the halt-again claim both fail.
  Fixed: fresh-spawn preparation keeps a latest round whose verdict is a reject or a superseding approve, and the gate reads the latest round before opening a request, with all four cases stated.
- **F4 (MEDIUM), supersede bound.** Fixed: halted at the cap means `blocked`, cap rejected rounds in the store, and the cap's reject as the latest verdict; approve checks no caller identity, so that state is the only gate.
- **F5 (MEDIUM), refusals untested.** Fixed: a new acceptance criterion covers supersede refusals, resume without approval, and session loss or deadline after a reject; the gate criterion covers an attach between rejects.
- **F6 (MEDIUM), failed liveness read.** Fixed: a failed read counts as not live, holds with a Warn, and is never returned as a gate error.
- **F7 (MEDIUM), false constraint.** Fixed: the constraint now covers only the orch session, plus batten sending into no session.
- **F8 (LOW), nested-pair timed pass.** Fixed: Decision 4's Bound states it; Open risks keeps only the upward-message consequence.
- **F9 (LOW), re-argued alternatives.** Fixed by cutting, not moving: both arguments already sit in the support log's Rejected alternatives, and the requirements they carry stay in Scope Out and Constraints, so the completeness-before-leanness test holds.
- **F10 (NIT), off-by-one comparison.** Fixed: the rationale now says three rounds give the writer two rewrites.

## Independent pass over the current artifacts

- The orch is a reed strand in the prime (`internal/orchcli/start.go` adds it under `agentname.RoleOrch`), so Decision 4's liveness seam over the parent worktree's reed session has a real target for a prime parent.
- Each new edge states its bound.
  The reject-can-halt edge is limited to three rejected rounds and to `blocked`.
  The supersede path depends on halted state alone.
  The engine-guard lift passes only on an approve, an expiry, no reviewer, or no open request.
  The hold ends at `parent_review_wait_min`.
  Batten's untimed halted wait is stopped by `lyx batten pause`.
  The legacy seed fallback covers only seeds that already exist.
- The new terminal `GateResult` only fails an entry sooner; it lets nothing through, so it needs no further bound.
- Rationale paragraphs that remain each state the chosen option's own reason in a clause, and do not re-litigate the alternatives.

Nothing further rises to a finding.
