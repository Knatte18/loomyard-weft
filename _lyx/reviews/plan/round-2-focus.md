---
round: 2
focus:
  - "Test budget: `cmd/lyx/testbudget_test.go` fails when a package's top-level `func Test` count exceeds its row in `cmd/lyx/testdata/test-budget.yaml`, or when a package has tests and no row, in the untagged tier. Settle whether every card that adds or deletes top-level tests in `internal/reedengine`, `internal/vscode`, `internal/hubgeom`, `claudeengine` or the new `internal/segmentcolor` lists that file among its targets with a row matching the cumulative count."
  - "Tagged-tier compile coverage: a card that deletes a field or changes a signature must list every caller in files under `-tags tmux` and `-tags integration`. Settle whether card 6 (`Geometry.RepoName` removal) and card 15 (`ideengine.Spawn` return change) name every `_test.go` file that still sets the field or calls the old form, so `go vet -tags tmux ./...` and `go vet -tags integration ./...` in the plan's `## verify:` compile."
  - "Attack-surface question for cards that add a routing edge or stored tmux option that is later expanded into a command: settle whether the card's `**Intent:**` states who writes the value, what it holds, and what an empty or stale value does."
  - "Ordering sources: where a card orders items by a timestamp, settle whether the source survives a checkout, rebase or re-materialized worktree, or the Intent states the bound."
---

Focus entries are places to look, not findings; each is judged against the rubric afresh.
