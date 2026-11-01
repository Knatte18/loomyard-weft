---
round: 1
focus:
  - "internal/landingshed/finalize.go: does the fix-review loop terminate at finalize_fix_review_rounds on every path (reject, fixer spent, resume after halt), with the unresolved findings reaching the halt, and is the fixer's turn cap reset per round as the commit message claims?"
  - "internal/landingshed/fixaccept.go and internal/loomcli/circling.go: after `lym loom circling accept` at the Finalize halt, does the resume land the change as it stands with the findings appended to the change description, and does a plain resume without accept rerun the fix reviews afresh? Check state persistence and the accepted-then-changed-tree case."
  - "Reroute removal (Deps.RerouteReviewed, HandsBack reroute-cap clause, on_reroute in tapestry, darn, sampler and pawl recipes): is no caller, test, spec or stencil still referring to the removed path, and does Finalize never reroute in any recipe?"
  - "Behavior change against the board entry: a spent fixer still halts as before; an over-cap fixer change always reaches the fix review in every recipe; Parent-Align and Code-Review are not rerun."
  - "contracts/specs, stencils and docs/overview.md edits: do they match the code's new behavior, and does the changed fixer and orch-role stencil wording drop or weaken any rule the entry did not ask to change?"
  - "Test economy in finalize_gate_test.go, fixaccept_test.go, circling_test.go and the wiring tests: new tests duplicating existing coverage, and new Finalize behavior (loop cap, escalation, accept path) left uncovered."
  - "Summary claims in _lym/landing/summary.md, when present, against the diff."
---

Seed focus for round 1, derived from the board entry in the decision record and the commit range diff.
