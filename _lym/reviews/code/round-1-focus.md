---
round: 1
focus:
  - "Scope against the board entry: it asks the gate to skip the fail-on-base check for a non-bug task, or let the Sketch or plan declare that kind, and to reword the finding. Settle whether the parent-recorded `No bug fixed` title in `internal/loomshed/redfirstgate.go` `exempt` meets that, and whether anything else the diff changes goes beyond it."
  - "Correctness of `exempt` in `internal/loomshed/redfirstgate.go`: the title loop compares case-insensitively with a prefix match for both `gateslot.NoTestSeamTitle` and `gateslot.NoBugFixedTitle`. Settle whether the no-test-seam behaviour is unchanged and whether the log line still reads right for either title."
  - "Findings wording in `redFirstFailure` and the `redFirstCall.run` green message in `internal/gatecli/redfirst.go`: check the format verbs against their arguments, and that the way forward for a real bug still tells the writer to make a test fail on the base. Settle whether the new `none of them fails before the change` sentence still says what the gate checks."
  - "Whether the hold lines (`redFirstHoldLine`) and the findings for the no-test and unjudged exits are unchanged in meaning after `noSeamAsk` became `parentRecordAsk(title)`."
  - "Docs and contracts landing with the change: the `darn-recipe.yaml` comment, `loom-discussion-spec.md`, the `darn-template-write.md` stencil, `docs/overview.md`, and the `doc.go` files of `gatecli`, `loomshed` and `shedrecipe`. Settle whether each states `No bug fixed` the same way, whether the stencil drops or weakens any existing rule, and whether another doc or stencil that names the `No test seam` way forward is left stale."
  - "Tests in `internal/loomshed/redfirstgate_test.go` and `internal/gatecli/redfirst_integration_test.go`: the changed green-case assertions must still check what their names say with no check loosened, and the new `No bug fixed` row must be covered without duplicating the existing `No test seam` row. The reach report lists no mapped test for `redFirstCall.run`, so check that the integration test covers the new message."
  - "Commit message trailers: the `Root-Cause:` trailer must name the defect the diff removes, given the entry is a tests-only task with no bug to fix. Check that no claim of agreement or improvement is made without figures."
---

Seed focus for round 1.
The answer key is the decision record alone, whose `## Goal` holds the imported board entry and which has no added entries yet.
