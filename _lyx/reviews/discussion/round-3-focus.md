---
round: 3
focus:
  - "Resume behavior of Finalize: check whether the decision record's D4 and R2 describe what a resumed Finalize computes, given that `Fabric.Merge` and `Fabric.MergeIn` in `internal/fabricengine/merge.go` merge the warp checkout alone and never the weft; settle whether the stated bound (duplicate versus lost notice) and the base of the config diff are correct for squash and non-squash landings alike."
  - "Failure semantics of the config-diff notice: check whether D4 and its acceptance criteria treat a failed diff read consistently with loom's rule in `PATTERN-refusal-way-forward` that a guard not protecting correctness warns and records, and with `Finalize.Call` mapping git faults to a returned error rather than Stuck; settle what the landing does and what the hub is told on a read failure."
  - "Bound of D5 and D7 per verb: check, for `add` (which carries skip options), `Checkout(l, branch)` and `Reconcile(l)` (which take none), whether the decision record states what each can fork past, what happens with an unreachable origin or a hub with no weft origin remote, and the way forward; settle whether the stated bound holds for each verb."
  - "Skip options in D6: check whether the live-pair rule states what happens under `SkipPush` and `SkipGit`, where `Add` does not run the leftover probes and D5's resolver skips origin."
  - "Assertion coverage of the stated behaviors: check that each behavior a Decision states (a live pair whose origin has no warp branch, `reconcile` adopting an origin weft covered by an archive tag, `add` under a skip option with an origin weft present, `.weft/` on the origin-adopt paths) has a clause in some acceptance criterion."
  - "Placement of plan-actionable rules: check that any ordering constraint the plan must obey (for example that the live-pair decision and every refusal precede the first creation) lives in a Decision, not only in the optional Notes for the plan writer."
---

Entries restate their own sites and claims so the reviewer needs no prior-round material.
