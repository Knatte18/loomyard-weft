---
round: 4
focus:
  - "Per-package test budget: `cmd/lyx/testbudget_test.go` counts top-level `func Test` functions per package against `cmd/lyx/testdata/test-budget.yaml`. For every card that adds a top-level test function or file to a package, settle whether the same card raises that package's row, or folds the behavior into an existing test."
  - "Pinned method lists in `cmd/lyx`: `gitrepoboundary_test.go` pins the `*Repo` methods that call `r.run(` or `r.runChecked(` in `gitrepoPinnedRunBoundMethods`. For every card that adds or removes a method in `internal/gitrepo`, settle whether the card lists that test file and states the list change."
  - "New required seams and shared fixtures: where a card makes an entry require a new seam through `requireSeam`, settle whether every shared fixture and conformance row that builds that entry (`internal/shedrecipe/fixture_test.go` `newTestEnv`, `conformance_test.go`, `entries_gate_test.go`) is in a card's targets and fills the seam."
  - "Pending-findings guard and its refusals: settle whether each refusal of the archive guard that `reset --to start` and `run --fresh` share names a way forward that makes progress for the reset caller, and not a step back to the refused verb, including the refusal for recorded starts that share no single oldest commit."
  - "Changed-text sites in `internal/webstercli`: settle whether every long-help or message sentence that a card's behavior change falsifies (the `runCmd` mismatch sentence, the `resetCmd` standalone and 'changes no other webster state' sentences) is a target of the card that changes the behavior."
  - "Card sizing: settle whether any card bundles independently reviewable units, or adds exported symbols called from another package without a card of their own, per the granularity rule."
  - "Uncommitted-paths reader: where a card's result lists the worktree's dirty paths from `internal/webstercli`, settle whether a named, exported seam over `Geometry` supplies it, since `Geometry.git` is unexported."
---

The entries name where to look; none caps a severity or presumes an outcome.
