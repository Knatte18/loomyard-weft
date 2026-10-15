---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: contracts/stencils/twill/twill-directive.md:55
    summary: The critical-fixer judge step renders at Discussion-Write too, breaking the record's superseding decision that it applies at Plan-Write only and that Discussion-Write keeps today's twill fixer steps unchanged
    fix: Make the judge step a host-chosen part, as the writer-fork and relaunch parts already are. Move today's critical-fixer steps 4 and 5 out of twill-directive.md into their own part stencil, filled with governing_input and dispute_evidence, and register it in contracts/stencils/stencils.go. Restore today's text from main as a second part, step 4 "Judge every finding with the same skepticism, and merge duplicates across lenses." plus {{.rejection_rule}}, and step 5 "one line per finding, with its lens and whether it was fixed or rejected and why". Directive in internal/twilldirective/twilldirective.go fills exactly one of the two parts into an optional marker. In internal/loomengine/twill.go, keep GoverningInput and DisputeEvidence on planLensDirective, and give discussionLensDirective back its RejectionRule "Fix the record, or reject the finding with a one-line reason." in place of its GoverningInput and DisputeEvidence. Update contracts/stencils/markers_test.go, contracts/stencils/claims_test.go and internal/twilldirective/twilldirective_test.go to match, revert the discussiontable_test.go rows that pin the critical fixer at Discussion-Write (keep its re-prompt and D5 boilerplate rows), and revert the critical-fixer wording for Discussion in docs/overview.md and internal/twilldirective/doc.go. Keep the D3 re-prompt rule and the D5 fork boilerplate host-neutral at both hosts, as the record's D3 and D5 still require.
---

### [BLOCKING] The critical fixer renders at Discussion-Write against the record's superseding decision

**Location:** `contracts/stencils/twill/twill-directive.md:55`, `internal/loomengine/twill.go` (`discussionLensDirective`), `internal/loomengine/discussiontable_test.go`, `docs/overview.md`, `internal/twilldirective/doc.go`

**Issue:** The decision record's last entry, "Added after Discussion (operator, 2026-10-15): Critical fixer only at Plan-Write", supersedes the entry before it and decides that the strict critical-judge fixer applies at Plan-Write only, with Discussion-Write keeping today's twill fixer steps unchanged.
D6 and the acceptance criteria still name both hosts, but the operator's later entry overrides them, and its rationale is the Discussion fork0 bench (2026-10-15), where the critical fixer did worse at Discussion.
The diff follows the superseded D6: step 4 of the shared directive is the critical fixer for every host, `discussionLensDirective` drops its rejection rule for `GoverningInput: "the board entry"` and a Discussion `DisputeEvidence`, `TestDiscussionTable` pins that text into the Discussion lead prompt, and `docs/overview.md` says the Discussion lead "judges each finding as a critical fixer".
The plan's overview states the same superseded scope ("The twill directive's judge becomes a critical fixer at both hosts").
The rest of the run matches the goal and the record: fork0 at Plan-Write only (D1 to D4), the host-neutral re-prompt rule and fork boilerplate (D3, D5), the `branch-coverage` lens and the `forced-edits` and `criteria-coverage` sentences (D7 to D9), and an unchanged Plan-Write gate.

## Overlap

`lyx loom peers` lists no parent commits.

- `tokencount-fixes` (row 17, own row 22): changes `tools/tokencount/` and `cmd/lyx/testdata/test-budget.yaml`; this run touches neither, so there is no overlap.
- `shed-tackler` (row 7): no changed paths yet; its title and board brief move the Go driver and tackler into shed and do not touch the twill directive, the loom hosts or the lens fans.
- `reed-launch-fixes` (row 0, unknown order): no changed paths and no decision record; its scope is reed launch lines and window focus, with no overlap.

No landing order applies, since no peer overlaps.
