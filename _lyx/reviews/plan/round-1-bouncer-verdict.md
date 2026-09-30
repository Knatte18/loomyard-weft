---
verdict: APPROVED
rationale: "The fixer resolved all ten round-1 findings in the current plan, and a check of the code turns up no new defect: card 15 now lists every ReadApproval/ChildApproval user, card 12 edits publish_verify_test.go, cards 10 and 11 fill the gate's required landing fields in both fixtures, card 16 carries all four stale doc statements, and every Decision D1-D9 and every Constraint is carried by a card."
---

# Plan-Review round 1 — judge verdict

The review file carries no `## Focus departures` section, so there is nothing to ratify or reject.
The reviewer answered all six focus directives, and each finding it raised is in scope for the rubric.

## Round-1 findings against the current plan

The plan files were edited after the review (fixer report, 18:47).
I judged the current plan and checked each fix against the working tree.

- **F1 (card 15):** `grep ReadApproval|ChildApproval` finds 15 files.
  Three of them (`landingshed/approval.go`, `publish.go` and `approval_test.go`) hold `landingshed.ReadApproval`, not the seam.
  Card 15's Edit list covers the other twelve, and its Intent and ImpactSummary name `entries_batten.go`'s `requireSeam` call.
  Resolved.
- **F2 (card 12):** card 12 now edits `publish_verify_test.go`, the only file outside `publish_test.go` that uses `newApprovalFixture`.
  `awaitingOrCancelled`/`checkApproval` appear only in `publish.go`, `publish_test.go` and the deleted `publish_awaiting_test.go`.
  Resolved.
- **F3 (card 10):** `internal/shedbuild/fixture_test.go` is on card 10's Edit list, and its Intent fills `ApprovalPath`, `RejectionPath`, `TaskHead`, `ReworkSpec` and the `Rework` seams.
  Resolved.
- **F4 (card 11):** the loomrecipe `testLandingDeps` now fills the three gate fields.
  The only production `loomrecipe.New` callers are `loomcli/arm.go` and `sharedbootstrap.go`, which card 7's `landingDeps` change and card 11's `wiring.go` change cover.
  Resolved.
- **F5 (card 16):** `docs/overview.md:363-364` and `:410` and `loom-status-spec.md:43` are the only doc statements that tie approval or the awaiting hand-off to Publish, and card 16 names each one.
  The `ly-drive` line "blocked at Publish" is an example label that remains valid.
  Resolved.
- **F6 (card 6):** the runlevel case now seeds a finished run and asserts the archive and removal before Master spawns.
  Resolved.
- **F7, F8 (card 9):** step 2 names the enumeration source, and step 1 closes the post-`ClearRejection` crash window, with both cases in the test list.
  Resolved.
- **F9 (card 12):** the ImpactSummary names the four consumers of Publish's `Awaiting` and states that batten is unaffected.
  Resolved.
- **F10 (card 1):** the caller count is corrected.
  Resolved.

## Rubric pass over the fixed plan

- **Decision-record fidelity:** D2 is carried by cards 7, 11, 12 and 16, and D3 and D9 by card 11.
  D4 is carried by cards 1, 2, 13 and 14, D5 by cards 3, 4, 5, 8, 9 and 11, D6 by card 6, D7 by card 12, and D8 by card 15.
  Each Constraint has a carrier: the registry and `Name*` constants (10, 11), the told scratch and durable paths (2), the fabric commit and read seams (5, 11), planparser as sole parser (3, 4), and CLI/Cobra plus help tests (13, 14).
  The docs Constraint is carried by the recipe header (11) and by overview, ly-drive and the status spec (16).
  Card 8's `Interactive: false` means autonomous per `shuttleengine.Spec`, still a tmux session, so the no-`claude -p` Constraint holds.
  No card adds unlicensed scope.
  The coverage file and `rejected_at` are implementation detail of D5's round record.
- **Granularity, ImpactSummary and `Custom`:** there are no `Custom` cards.
  Every `Edit`/`Delete` card's ImpactSummary states a blast-radius conclusion.
  Card boundaries follow independently testable units.
- **Verify coverage:** `go test ./...` and `go test -tags integration ./...` cover every targeted package, and `smoke` is only vetted.

One residual note, not a finding.
Card 10's `prGateEntry` test needs `entries_simple_test.go`'s `validLandingDeps` to gain `ApprovalPath`, `RejectionPath` and `TaskHead`.
The card doesn't say so, but the file is on its Edit list and the test it describes ("builds a `*landingshed.PRGate`") cannot pass without them, so the implementer has no other reading.
