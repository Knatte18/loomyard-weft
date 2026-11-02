```yaml
lens: fix-consistency
outcome: reported
```

I checked whether the round-1 change set agrees with the parts of `decision-record.md` it touches: sibling decisions, Scope, acceptance criteria, Constraints and the support log.

- **LOW** — Scope "Out", the bullet on Darn-Write's `clean` and `description` gates, against the new D2 paragraph that starts "Every other gate entry of a row that carries a fixer entry is wrapped to wait only".
  - Summary: Scope still lists the `clean` and `description` gates as Out with no qualification, but D2 now wraps both of them in a wait on a live fixer. A reader of Scope would not expect those two entries to be wrapped at all.
  - Evidence: Scope says "Darn-Write's `clean` and `description` gates: a leftover file is the writer's to keep or delete…". D2 says "On Darn-Write this keeps the `clean` entry, which goes first, from failing on a live fixer's uncommitted edits." The Scope bullet should say these entries only wait on a live fixer and never get a fixer of their own.
- **LOW** — D8's last bullet, "The fixer's time shows in the gate entry's gate span", against D2's new wait-only wrap.
  - Summary: on a restart, the finalize evaluation does its waiting inside whichever entry it calls first, which on Darn-Write is `clean`. The fixer's time then lands in that entry's gate span, not the fixer entry's, so D8's claim about where fixer time shows is now stale.
  - Evidence: D2 says "whichever entry it calls first does the waiting". `evaluateGate` records each entry's span around its own closure call: `run.gateSpans[i] = append(... GateSpan{Kind: GateSpanGate, Start: closureStart, End: closureEnd})` at `internal/shuttleengine/wait.go:1665-1667`.

I found no other mismatches between the changed text and the parts it touches.

I made no further tool calls.
