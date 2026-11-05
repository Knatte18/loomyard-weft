---
round: 2
focus:
  - "Cards 14 and 15 (`_lym/plan/14-beam-pr-status-close.md`, `_lym/plan/15-beam-ready-refusals.md`): settle whether `lym beam ready` can pass its pull-request-mode refusal on a hub with no GitHub origin, with no usable token, and for a beam with no commits beyond its parent, under each parent mode, given that `pullrequest.Open` refuses a non-GitHub origin and `landingshed`'s resolver reads rules only when the origin facts pass."
  - "Card 10 (`_lym/plan/10-beam-await-entry.md`) and `awaitRun.accept` in `internal/pawlengine/beamawait.go`: settle whether a crash between accept's removals, followed by a resume from a halted status, leaves a state that one of the card's entry detections settles, with the dobby's land episode already recorded."
  - "Cards 12 to 15: settle which run states `judgeAwaitingRun` passes at Beam-Await, given that a producer error persists as `failed` in `internal/shedengine/run.go` and that `TestDescribe` rows enter only reachable states."
  - "Cards 1, 5, 7, 12 and 15: settle whether text stating the old hold boundary, the Parent-Align merge-in at landing, the PR-Review reroute or the BeamDescribe row (doc comments, `doc.go` files, cobra help) is rewritten by some card, and whether the retired-row mechanism the plan adds is reconciled with `loomrecipe.RetiredRows` and the docs that describe it."
  - "Card 3 (`_lym/plan/03-pr-merge-state.md`): settle whether required status contexts reported through the commit-status API, as well as check runs, are read, since the cause text of `lym beam pr status` and the blocked cause list each required check."
---

Focus entries name where to look; the reviewer still reads the whole plan.
