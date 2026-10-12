---
round: 2
focus:
  - "In `_lyx/discussion/decision-record.md` D8 and D5: settle whether every route the decision record gives an operator decision on a task a run holds can be carried out given `internal/boardengine`'s run lock, which refuses any write on a held entry that touches a field other than status, and whether D5's bound on an auto-landing agent approval still holds with that route."
  - "In `_lyx/discussion/decision-record.md` D3 and D6: settle whether the in-flight peer set is defined once, against the board's run status shape `<state> · <producer>`, and used identically in D3's overlap check, D6's `show` refusal and D6's Bound, including peers in the awaiting and blocked states."
  - "In `_lyx/discussion/decision-record.md` D5 and D7: settle whether each stated bound names a limit that exists in the code, such as `merge_auto_accept` in Merge-Review and the `ConfigSpec` categories in `internal/loomengine/config.go`, rather than an assumed one."
---

Focus targets the design-level premises behind the BLOCKING and MEDIUM findings; lower-severity items need no steering.
