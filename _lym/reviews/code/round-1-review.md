---
verdict: APPROVED
---

No finding survives consolidation.

The new `lym-plan-edits-before-gate` entry in `internal/heddleengine/shipped.json` sits after `lym-batching` (order 20 against 10), targets `claude-file` alone, and is worded as a command covering all four steps.
It names all four forbidden patterns: one file read per turn, one file read in slices over turns, an edit, a read and another edit spread over turns, and a gate run between the edits of one change.
`ShippedEntriesArePinned` pins slug, order position and target for both shipped entries without duplicating another test.
The other tests that read `shippedStore.Ordered()[0]` still resolve to `lym-batching`, and the render and list tests derive their expectations from the layer.
The diff touches no heddle-side code and no appended system prompt, which keeps it inside `PATTERN-lym-instructions-in-code`.
The stencil removals match the batching lines `bf00a043` added, and the card-specific parts stay: the spans a Webster card names and a darn commit as the unit of one change.

## Rejected

- Fork finding F1 (code-affected-set, MEDIUM scope): `contracts/stencils/webster/webster-body-implementer.md:58` ("Read every hit's span in one message, then edit every hit in one message") is not a `turn-batching` line.
  `git log -S` traces it to `dcbe858e`, the retired-name sweep, so it is the card-specific sweep instruction the record says to keep, and `bf00a043` did not add it.
- Fork finding F2 (code-affected-set, LOW consistency): the rendered burler explore text in this review's prompt still carries "in one message".
  That text is rendered from the hub's stencil store when this round started, not from the diff's `contracts/stencils/burler/burler-step-1-explore.md`, and no Go source holds a second copy (grep of `internal` for "in one message" in non-test Go finds only the edit directive, the twill directive and the shell-guard edit rule).
  The diff is the one home of the stencil, and the prompt is not the subject.
- `contracts/stencils/edit/edit-directive.md:8-10` and the fork-spawn lines in `burler-fix-self-check.md:10`, `twill-directive.md:13` and `webster-template-master.md:126` remain by design: the record keeps `edit-with-edit-tool` as a separate rule, and the fork-spawn lines are not read or edit batching.
