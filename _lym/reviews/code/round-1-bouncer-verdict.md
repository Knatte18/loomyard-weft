---
verdict: CONTINUE
rationale: "Round 1 carries F2, a design-class MEDIUM (Main's snapshot, run and check wiring in internal/testkit/tmuxkit/tmuxkit.go has no test), which is gating; no BLOCKING finding and no relabel. F1 is decision-class MEDIUM and non-gating, deferred to the parent."
---

F2 gates the round: the wiring `Main` performs is the behavior the change exists for, and nothing pins it.
F3 to F6 are design-class LOW or NIT and do not gate; they ride along with the F2 fix.
F1 is a `decision` finding about two unrecorded departures from the board entry; its fix is the parent's `lym loom decision add`, not a code change.
No finding was relabeled.
