---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: cmd/lyx/testdata/test-budget.yaml:55
    summary: internal/loomcli row is 183 but the package holds 184 top-level tests, so TestTestBudget_PackagesStayWithinBudget fails Publish
---

### [BLOCKING] `internal/loomcli` test-budget row is one short after the merge-in

**Location:** `cmd/lyx/testdata/test-budget.yaml:55` (`internal/loomcli: 183`).
**Issue:** `TestTestBudget_PackagesStayWithinBudget` in `cmd/lyx` fails, the Publish verify failure this round exists to fix; I reproduced it with `lyx gate test ./cmd/lyx`.
The failure reads `internal/loomcli: 184 top-level tests, over its budget of 183`.
The cause is two independent raises of the same line that merged without a conflict.
Card 7 (`22e9d96b6`) added `TestWire_DiscussionTableEvaluatesToExpectedShape` and raised the row `182 -> 183`, as the plan's `test-budget-trajectory` decision prescribes.
Main (`b6e1c3601`, brought in by merge `5468bc704`) added one `internal/loomcli` test of its own and made the identical `182 -> 183` edit.
Git took the identical edit once, so the merged row counts one added test where two exist.
The other rows the branch raised (`internal/loomengine`, `internal/loomrecipe`, `internal/shedrecipe`) still hold; only `internal/loomcli` is over, as the gate run shows.
**Fix:** raise the `internal/loomcli` row to `184` in `cmd/lyx/testdata/test-budget.yaml`, in a commit of its own on this branch, then rerun `lyx gate test ./cmd/lyx`.
No test is added or removed: the branch's one `internal/loomcli` test is the plan's and has no existing test that covers it.

## Review of the branch's own range

The range is `b6e1c3601..HEAD`, the branch's commits after its parent `main`; no `_lyx/loom/rework/` round exists.
I read the production diff of cards 1 to 9 against their cards and the plan's shared decisions and found no further defect:

- `seatengine` optional values: `Table.Optional`, `requiredValues`, `Prompts` and `composeSeatPrompt` match card 1, and `Engine.startSeat` calls `composeSeatPrompt` with the table's list.
- `shedadapters.NewMultiLLMProducerSourced`: the table is built once per `Call`, before the probe, and the preparation runs on the fresh path only, after the probe and before the archive.
- `loomengine` config: `discussion_producer` and `discussion_advisors` are validated at load, carried whole through reconcile via `ConfigOpenMaps`, and each new refusal names its way forward, has a row in `contracts/specs/refusal-spec.md` in the same commit, and is reached by a test row (`PATTERN-refusal-way-forward`).
- `DiscussionTable`, the two stencils, `roleOpeningStencils`, `shedfake.SeatRunner.stampGate`, `discussionSeatsEntry`, `loomrecipe.applyDiscussionProducer` and the `loomcli` wiring follow their cards.
- The coverage-guard bridge is closed: `DiscussionSeats` is in `RecipeEngines` and no longer in the allowlist.
- Docs landed with the code: `doc.go` of `seatengine`, `shedadapters`, `shedrecipe` and `loomrecipe`, `docs/overview.md`, and the two specs.
- Test economy: no duplicate or per-helper test found; the new loomcli, loomengine, loomrecipe and shedrecipe tests each cover a behavior the branch adds.
