---
verdict: APPROVED
rationale: "All seven round-1 findings (F1-F7) are fixed in commits 4d8702568..48b60f53a and verified in the code; the diff over merge base 2cb4ac68 (product.parent main) matches cards 1-16, card 12's Delete group passes assert-no-callers, and go build, go vet, go test, go test -tags integration ./... and go vet -tags smoke ./... all pass."
---

# Webster-Review round 1 — Bouncer verdict

## Review range

`product.parent` is `main`, read from `_lyx/shed/publish-review-fixback/status.json`.
The range is `git diff 2cb4ac68..HEAD`, which now includes the fixer's seven commits on top of cards 1–16.

## Substrate

I re-ran the plan's `## verify:` list in full at HEAD `48b60f53a`.
`go build ./...`, `go vet ./...`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` all pass.

## The round-1 report

The report is sound, and I ratify its findings and its "settled without a finding" section.
It has no `## Focus departures` section, so there is nothing to ratify or reject there.
Its per-card mechanical check is correct.
Card 12's `Delete` group is the plan's only mechanical target.
`publish_awaiting_test.go` is gone, and no reference remains to `awaitingPRURL`, `publishOutcomeFor`, `awaitingOrCancelled`, `checkApproval` or `newApprovalFixture`.
Card 16 is `Prosa`, and no card has a `Rename` group.

## Fixes verified

- **F1**: I scanned every added Go comment line for mid-sentence breaks and for two sentences on one line.
  Every remaining in-sentence break follows a comma and a coordinating conjunction.
  Two of these, the file comments of `internal/loomcli/approve.go` and `reject.go`, break before a compound predicate (", and never resumes the run itself").
  That is a phrase boundary, not an independent clause, so strictly it misses the rule.
  It does not reach the fixed-column mid-phrase wrapping F1 described, and it does not block.
- **F2**: `PendingRejection` now cites `battenshed.ChildDecision`.
  A tree-wide sweep finds no `ChildApproval`, `ReadApproval:` seam, `approvalActed` or `-approval-acted` outside `_lyx`.
- **F3**: `PRGate.done` consults `cancelErr`, and all three Done exits route through it.
  `TestPRGate_CancelledBeforeDone_IsError` pins it.
- **F4**: The rework stencil now requires at least one new card for every finding, and it says that a round with no new card is rejected.
- **F5**: `finish` says it runs step 6, and that the absent-record crash path calls it with `clear` false.
- **F6**: `ReadDecision`'s comment now limits the approval-first order to a race between the two verbs, where it matches the gate's behaviour when both records match.
- **F7**: The batten paragraph in `docs/overview.md` names both `approve` and `reject`.

The fixer also corrected `internal/landingshed/doc.go`'s package line to "three general producers".
The diff had made that line stale, and the correction is right.

## Independent pass

I read the Go-owned logic against the cards and found no new defect:

- the `PRGate.Call` decision table against card 7;
- `prRework.Call`'s step order, crash re-entry and round numbering against card 9;
- `reject`'s refusal table and its PR-Rework blocked path against card 14;
- `approve`'s move to the gate against card 13;
- Publish's refresh-and-Done split against card 12;
- batten's decision identity and marker rename against card 15;
- the recipe rows, entries and wiring against cards 10 and 11.

The rejection record lives under the ephemeral scratch dir.
It therefore stays outside the rework commit's pathspec, which covers only the plan dir and `_lyx/loom/rework`.
