---
round: 2
focus:
  - "Where the plan renames an unexported identifier to an exported one (for example the `clock` interface in `internal/shuttleengine/wait.go`), check that every in-package file naming the type, test files included, is among the owning card's Edit targets, so `go test` of that package still compiles."
  - "Where the plan splits or bundles the hold in `Wait` with the notifier seam (`SetNotifier`, `hold.go`, `RunState.NotifiedOffset`, `RunState.StrandName`), check that each independently reviewable unit has its own card, that every cross-reference and the Card Index agree after any renumbering, and that the plan states which events-file offset an expired-shell hold records and what happens when persisting it fails."
  - "Where a card edits the shed driver stencil's messaging rules, check that the statement it adds has a stated placement that does not contradict the stencil's existing verbatim parent-notice rule."
---

The focus entries target the plan's structure and its internal consistency after the round-1 fixes.
