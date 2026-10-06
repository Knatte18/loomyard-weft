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

- **Decision:** test files are named as they stand on the current tree; once `test-suite-prune-core` and `test-suite-prune-rest` land and the pair is synced with main, the test targets are re-validated and re-planned only where a prune renamed or deleted them.
- **Rationale:** recorded post-Discussion decisions (operator and parent, 2026-10-06).
- **Applies to:** every card's test targets

### Decision: cli-drive

- **Decision:** each fix's regression drives the `lyx` cobra command (`lyx webster validate`, `begin-batch`, `record-batch`, `recover-batch`, `rebaseline`, `run`, `reset`) through `internal/webstercli`'s tests; Publish (card 12) is driven through its real producer over a hubforge hub with the real fabric closure.
- **Rationale:** card 12 departs from the decision record's "driven through the `lyx` CLI command" for D8, whose acceptance item is the push-rejected stuck reason alone: no loomcli test steps the Publish row today, so driving it through `lyx loom step` needs a new loom-step harness, which is new test infrastructure outside this task.
  The stuck reason is verified at `landingshed`'s integration tier over a hubforge hub with a real fabric push and remote-read closure, whose resumed half also reaches the GitHub step that only that tier fakes, and the production closure's real wiring is driven in loomcli's `landingdeps_integration_test.go`.
- **Applies to:** cards 1–8, 11, and card 12 as the rationale states

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
