---
round: 1
focus:
  - "Test economy, cards 3 and 4: card 4's new `TestNewDiscussionGate` subtest, `TestValidateDiscussionCmd` `Findings_RelevantMalformed` and `TestGateParity_DiscussionGate` `Stuck_RelevantMalformed` each assert the same stray-line finding through `ValidateLoomDiscussion`; settle whether an existing or sibling test already covers each, and whether `TestCheckRelevant`'s table already covers the check itself."
  - "Test economy, card 5 and 6: both edit `TestDiscussionTable`; settle that no planned assertion is on a stencil's static prose, and that the `finishes writing ` assertion tests a value rather than stencil wording."
  - "Fidelity to the decision record: check that every Decision D1-D6 and every Constraint is carried by a card, in particular D1's `ReviewedFiles` unchanged and the engine-model-choice constraint on `DiscussionTable`, D3's non-`Validate` placement, the doc-comment edits in the Scope list (`plan.go` header, `discussionparser/doc.go`), and that no card adds scope the record does not license (for example card 4's I/O envelope prefix change and `docs/overview.md` and PATTERN edits)."
  - "Card order and dependencies: cards 4-7 use `DiscussionRelevant` and `CheckRelevant`; confirm no earlier card needs a later card's symbol, and that cards 5 and 6 editing the same test are ordered consistently."
  - "Verify coverage: settle that every package a card targets (`discussionparser`, `loomengine`, `loomshed`, `shedrecipe`, `testkit/envkit`, `loomcli`, `twilldirective`, `contracts/stencils`, `cmd/lyx` test-budget) has its tests run by `## verify:`, including integration-tagged and smoke tests that card 4 edits, and that the section order matches the plan template."
  - "Edit-card glyph rules on card 4 and card 5: for each member whose parameters, results or receiver the Intent changes (`NewDiscussionGate`, `loomCLI.wire`, `validateDiscussionCmd`, `Env`, `Values`), check the re-sign arrow; check that file-path-only Edit targets (`internal/discussionparser/doc.go`, `internal/twilldirective/doc.go`, docs and PATTERN files) are not naming symbols in their Intent."
  - "Attack surface and reachability: card 3's path rule and card 4's `requireAbsRoot` guard; settle that each stated refusal has a named input and test that reaches it, and that no upstream check rejects the named input first."
  - "Granularity: card 4 bundles the gate function, resolver, `Env` field, CLI wiring, docs and many test edits; settle whether it is one independently reviewable unit or should split."
---

Seed focus from reading the plan and the decision record; no round has been judged yet.
