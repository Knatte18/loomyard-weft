---
round: 3
focus:
  - "Does `_lyx/discussion/decision-record.md` square the D2 way forward (commit or discard in the parent's code checkout with plain git), repeated in D4 and D5, with `contracts/specs/refusal-spec.md`'s `What a way forward is` list, given `PATTERN-refusal-way-forward` binds `internal/loom*` and `internal/shed*`? Settle whether a recorded decision covers any widening the text relies on."
  - "Does D2 state the route a Finalize re-step takes after an operator commits in the parent's code checkout, judged against `parentGuard.moved` in `internal/landingshed/parentguard.go`, `holdParent` and `rerouteIfMoved` in `internal/landingshed/stuck.go`, and Finalize's `on_reroute` row in `contracts/recipes/loom-recipe.yaml`, and does an acceptance criterion pin both the commit and the discard outcome?"
  - "When the parent's guard error carries `merge already in progress`, does D2's way forward name a verb and a way to see the parked merge, compared with `mergeWayForward` in `internal/landingshed/premerge.go`, so a stale record cannot wedge the run?"
  - "When the guard error carries `worktree dirty` together with another aggregated reason, does D2 say how the stuck reason and way forward treat the other reasons, and does an acceptance criterion cover the combined case?"
---

Focus entries target the open round 2 keys without capping severity.
