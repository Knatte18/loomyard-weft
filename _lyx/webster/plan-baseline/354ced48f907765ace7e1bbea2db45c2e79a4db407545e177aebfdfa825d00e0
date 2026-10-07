---
format: 5
approved: true
language: go
---

# Plan: bugfix-webster — plan-order gates and Webster, reset and Publish stops

Fix the Webster and plan-validity bugs the board entry `bugfix-webster` lists, per the decision record's D1–D10.
The Plan-Gate refuses a plan whose card order is not executable (a `Uses:` naming a later card's target, and a delete that runs before a later card removes its reference), so such a plan never reaches Webster.
Each reported stop either cannot recur or names a way forward that works the first time: an already-deleted Delete target at dispatch, a `delete-not-done` caused by a later card, a report left with no begin-batch record, a card edit inside a failed batch, a reset that left the remote task branch diverged, Publish's push-rejected halt, and the background-shell friction note; begin-batch's dormant model assertion goes.
Each fix carries a regression test that drives the real `lyx` cobra command wherever one exists.

## Card Index

1 — uses-later-target — Plan-Gate check refusing a Uses: that names a later card's target (D1)
2 — delete-before-reference — Plan-Gate check refusing a delete a later card's Edit code still references (D2)
3 — delete-target-gone — an already-deleted Delete target of an unbegun card is an advisory at dispatch (D3)
4 — drop-model-assertion — begin-batch stops asserting and injecting Master's model (D10)
5 — archive-unrecorded-report — begin-batch archives a report with no begin-batch record (D5)
6 — rebaseline-terminal-batch — rebaseline --card accepts an edit to a failed, dead or stuck batch's card (D6)
7 — delete-blocked-by-later-card — delete-not-done caused by a later card names it, and recover-batch refuses to loop (D4)
8 — background-shell-outcome — the background-shell note and warning state what happened and the run's outcome (D9)
9 — gitrepo-leased-update — leased remote branch update and remote-only commit listing in gitrepo (D7 groundwork)
10 — fabric-remote-only-commits — Fabric.RemoteOnlyCommits, the remote task branch's tip and the commits HEAD lacks (D7, D8 groundwork)
11 — reset-moves-remote-branch — webster reset moves the remote task branch under an ancestry gate and a lease (D7)
12 — publish-push-rejected — Publish's push-rejected halt names the remote tip, the count and the resume (D8)

## Shared Decisions

### Decision: targets-are-files

- **Decision:** cards name production code and tests by file path, not by member glyph.
- **Rationale:** several cards edit the same files (`beginbatch.go`, `doc.go`, `refusal-spec.md`, `verbs_test.go`); file targets keep every shared target a forward edge between cards and avoid member-versus-file containment overlaps, and the parent's post-Discussion decision asks for test files by path so the plan survives the two test prunes.
- **Applies to:** all cards

### Decision: test-targets-after-prunes

- **Decision:** test targets and named tests follow the tree after `test-suite-prune-core` (`dd8f4ffcd`) and `test-suite-prune-rest` (`959025744`) were merged into this branch; where a prune merged a named test into a table or scenario, the card extends that table's row or adds a row or step to it, and where a prune deleted or renamed a test file, the card targets the file that now holds the coverage, or creates one where no file does.
- **Rationale:** the parent's 2026-10-07 decision to re-plan only the cards the prunes touched; cards 1, 2, 4, 5, 6, 7, 8, 9, 11 and 12 were revised for it, card 3 and card 10 are unchanged.
- **Applies to:** every card's test targets

### Decision: cli-drive

- **Decision:** each fix's regression drives the `lyx` cobra command (`lyx webster validate`, `begin-batch`, `record-batch`, `recover-batch`, `rebaseline`, `run`, `reset`) through `internal/webstercli`'s tests; Publish (card 12) is driven through `internal/loomcli`'s real `wire()` seams and the production `landingDeps` closure, as the `wiring_*_integration_test.go` files drive their rows.
- **Rationale:** no loomcli test steps the Publish row today, and `lyx loom step` is not driven by a test that reaches GitHub.
  Card 12's rejected half halts before any GitHub call, so `wire()` reaches it with no new harness; its resumed half reaches the GitHub step that only `landingshed`'s integration tier fakes, so that tier also drives both halves of one scenario over a hubforge hub with a real fabric push and remote-read closure.
- **Applies to:** cards 1–8, 11 and 12

### Decision: dispatch-scope

- **Decision:** "unbegun cards" means the cards of every batch with no record in state.json; D1 and D2 at dispatch use `planglyph.ValidateDispatch`'s existing pending scoping, D3's downgrade applies only when at least one card is begun, and D4 checks a begun batch's own cards against the unbegun ones.
- **Rationale:** the decision record's D1–D4 scoping; the whole-plan entry points must keep refusing a missing Delete target.
- **Applies to:** cards 1, 2, 3, 7

## verify:

go build ./...
go vet ./...
go test ./...
go test -tags integration ./internal/planparser/... ./internal/planglyph/... ./internal/websterengine/... ./internal/webstercli/... ./internal/gitrepo/... ./internal/fabricengine/... ./internal/landingshed/... ./internal/loomcli/... ./cmd/lyx/... ./contracts/...
go test -tags tmux ./internal/webstercli/... ./internal/loomcli/...
go vet -tags llm ./internal/planparser/... ./internal/planglyph/... ./internal/websterengine/... ./internal/webstercli/... ./internal/gitrepo/... ./internal/fabricengine/... ./internal/landingshed/... ./internal/loomcli/...
