# Plan — round 2 fixer report

## Changes

- **F1** (fixed as proposed): in `04-implementer-verify-tiers.md`, the `renderForkWithPack` and `renderRecoveryWithUncommitted` Edit lines now carry ` -> ` and the new signatures with a trailing `verifyTiers []string`;
  Edit lines for `renderFork`, `renderForkPromptAndPack` and `renderRecovery` are added;
  the Tests bullet now names `renderForkPromptAndPack` among the callers that pass nil.
  Premise checked against `internal/websterengine/template_test.go` lines 332–364.
- **F2** (fixed differently, placement only): the D5 bound sentence in Card 3 sits at the end of the `loom-rubric-plan-review.md` bullet, where the "compiled rather than run is not a finding" rule is introduced, rather than after the Intent's first bullet (`RubricNotes`), which concerns struct fields and would separate the bound from the skip it bounds.
  Card 2's `## verify:` bullet gains the sentence that the plan writer may leave a compiled tier unrun, bounded as Card 3 states.
  Card 4's implementer-body bullet gains the sentence that a compiled tier is compiled, never run, by raw runs only, while every rendered card or batch gate step runs as written.
  Intent text only.
- **F3** (fixed as proposed): Card 3's `rubric_test.go` bullet now also says the doc comment of `rubricMarkerAllowlist` names `verify_tiers`.
  Premise checked against `contracts/stencils/rubric_test.go` line 18.

`lyx loom validate-plan` reports `ok`.

## Disputed

None.

## Deferred

None.
