---
verdict: APPROVED
---

The design covers every point of the board entry `loom-stencil-wording` and stays inside it.

- F6, rejection findings as a reviewer's claims: D1 rewords `loom-template-rework.md` and the Plan-Review rubric's rework answer-key bullet as the entry's Idea proposes.
  Its route for a Decision-overturning finding reuses the plan stencil's existing parent-recording route with no new verb, which gives the entry's "not on a par with the decision record" an outcome; the record discloses it as an auto-mode assumption.
- F15, F16, F17 and the same-kind rework gap: D2, D3 and D4, each matching the operator's decision to fix them as the agent proposed.
- Verify tiers from hub config: D5 to D7 move the Go tier text out of the plan template, the Plan-Review rubric and the implementer body into a `verify_tiers` list in `language.yaml`, as the entry's Idea names, and D5's rationale checks the shape against `dotnet-verify-protocol`.
  Adding the spec's `## verify model` follows the entry's goal, since it carries the same Go text to the same readers.

No other board entry is duplicated or contradicted:

- `dotnet-verify-protocol` and `dotnet-scoped-verify`: Scope Out keeps their machine-run gates, toolchain declaration and Go-coded gate generalisation out; `verify_tiers` is prompt guidance only.
- `webster-fork-stencil-fixes` depends on this task, and D6 edits none of the ranges it cites in the shared files (the plan's `## Rename mechanic`, the implementer's failure and report sentences, the spec's `Rename` row and per-language list).
  The record leaves the shape of that entry's rename-tool config to that entry.
- `orch-role-stencil-fixes`: the orch role stencil is in this run's Scope Out, and the driver stencil is in that run's Scope Out.

## Overlap with runs in flight

`lyx loom peers` lists no landed parent commits.

- `orch-role-stencil-fixes` (row 3, a tie with this run's `own_row` 3): no shared file.
  Its D4 rewords the orch's PR-Gate sentence so a small defect is fixed by the run's own agents through the reject, and its record asks that this run's D2 driver wording read the same way.
  This run's D2 says a fix to a halted run is made by the run's own agents through `reject` or a backward `goto`; the two agree, so the designs do not conflict.
  Landing order: a tie.
- `reed-add-parentless-placement` (row 7) and `creel-sandbox-suite` (row 3): no overlap.

Shared bookkeeping: a card adding tests to `internal/languageconfig` may edit `cmd/lyx/testdata/test-budget.yaml`.
