---
round: 3
focus:
  - "D11 (launch time): settle whether every figure the decision reads is held by the gate records and logs it names; `internal/gatecli/counters.go` `goJSONCounter.handle` writes only event output to the log buffer, and `RunRecord` in `internal/gateslot/runlog.go` has no per-package timing, so check where launch time comes from for runs before and after this task."
  - "D13 (retag Verify line): settle whether a hub that keeps `covers: [0]` on tier1 to tier3 in its gate config passes the card Verify line while the installed gate drops tier0 as covered, leaving retagged tier0 tests in no gate."
  - "D8 (entry gates and pauses): settle what `pause_after` and the entered-row mark do when a row is skipped by its entry gate or is routed back to through skipped rows."
  - "D14 (tree_scope migration): settle whether a second and later reconcile of a sparse hub file, where `tree_scope_verify` has been pruned, leaves operator-added `tree_scope` trees untouched."
  - "D5 and D13 (tag rename): settle whether `namesGateOnlyTier` in `internal/planparser/verifymodule.go` and the raw go row in `internal/shellguard/rows.go` still refuse gate-only tags once tests carry `tier2` and `tier3`."
---

Focus targets the places where round-2 fixes will be new text, plus the BLOCKING area.
