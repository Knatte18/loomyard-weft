---
verdict: BLOCKING
rationale: "F1-F10 are fixed and verified (build, vet, unit and integration tests pass on 21a1a9783), but four new doc comments in the range still break mid-sentence at a list-item or compound-predicate comma, against CONSTRAINTS.md's Comment Line-Break Convention: parity_test.go:546, wiring_commitstatus_test.go:411, wiring_rework_integration_test.go:217, gates_test.go:437"
---

The review range resolves from `_lyx/shed/rework-generations/status.json` (`product.parent: main`) to merge base `9ae5f9405`, covering the sixteen card commits and the eleven round-1 fix commits up to `21a1a9783`.
`go build ./...`, `go vet ./...`, `go test ./...` and the `integration`-tagged tests of `planparser`, `websterengine`, `loomshed`, `shedadapters`, `shedrecipe`, `loomcli`, `loomrecipe` and `landingshed` all pass on HEAD, and `gofmt -l` names no file in the range.

## Round-1 findings

The round-1 report carries no `## Focus departures` section, so there is nothing to ratify or reject there.
Its ten findings are sound, and each fix holds:

- **F1**: `roundFor` now decides on the highest round alone and returns its record as in flight unless its identity is among the committed rejections; `Call` re-keys an in-flight round whose identity differs from the pending one through `rekeyRound`, keeping `first_card` and dropping the class.
  `TestPRRework_SupersedingRejectionKeepsTheRound` and `TestPRRework_RecordlessLowerRoundIsNotResumed` pin both halves.
  One path was considered and not raised: a classless record that reached HEAD after an operator removed the rejection record by hand would now be re-keyed by a later rejection rather than opening a new round.
  That path needs a hand edit to a loom-owned record, and the plan's own bound for it (a classless record must not count as committed) still holds.
- **F2**: `writeRoundRecord` writes a sibling temp file and renames it, and an unreadable or undecodable highest record is a returned error with a way forward.
- **F3**: a classed in-flight record goes straight to `commitRound`; `finishRound` copies `coverage.md` before it writes the class, so a classed record implies the round is otherwise complete.
- **F4**: `PlanReviewSkippable` requires `ReviewExempt` over the working-tree plan too, and a working-tree parse failure is an error, so the Bouncer reviews for real.
- **F5**, **F6**, **F7**, **F8**, **F9**: fixed as the report asked, each with its test or doc change.
- **F10**: every site the report listed is rewrapped.

## New finding

### [LOW] Four doc comments still break mid-sentence at a non-clause comma

CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, with a further break only at an independent-clause boundary, per the `scribe:prose` rule it cites; that rule states that a comma joining a list item or a compound predicate does not trigger a break.
F10 targeted fixed-column wraps, and these four new comments in the range break instead at a comma inside a list or between a subject and its verb:

- `internal/loomcli/parity_test.go:546`–`547`: the list of cases breaks after its second item, `(stuck),`.
- `internal/loomcli/wiring_commitstatus_test.go:411`–`412`: breaks after `though each holds a file,` before the subordinate `whether ...` phrase.
- `internal/loomcli/wiring_rework_integration_test.go:217`–`218`: breaks between the subject `Env.SkipPlanReview, over a real fabric pair,` and its verb `answers`.
- `internal/loomshed/gates_test.go:437`–`438`: breaks inside a compound predicate, after `numbered cardNumber,` before `declares first_card`.

**Fix:** join each sentence onto one line, touching no other line.
