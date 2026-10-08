---
format: 5
approved: true
language: go
---

# Plan: plan cards name members as glyphs

Plan cards are to name the members they change as glyphs, so webster's guards, plan validation and every downstream agent work at member granularity.
Before the stencil pushes plans there, this plan tests loomyard's glyph chain over every member shape real plans name and fixes the defects that test exposes.
It then retires the two cross-card containment refusals and `handle-unreferenced`, and adds same-card redundancy, the `**Edit:**` re-sign arrow and caller coverage.
The plan stencil and the Plan-Review rubric change last, once every check they describe exists.

## Card Index

1 — glyphchain-plan-gate — commit the glyph-chain fixture module and the untagged plan-gate legs of the D1 test
2 — glyphchain-delta-legs — add the D1 test's integration legs: delta, handle binding, rewrite and done-checks, plus the rework-generations row
3 — rename-interface-method — derive a Rename's declaration for an interface method so it canonicalizes
4 — rename-multi-name-spec — derive a one-name declaration for a var or const Rename so a multi-name spec canonicalizes
5 — retire-cross-card-containment — remove containment-unit-overlap and containment-file-overlap
6 — retire-handle-unreferenced — remove handle-unreferenced so a handle needs no later reference
7 — redundant-package-target — flag a package self glyph listed beside a member or file of that package on one card
8 — redundant-file-target — flag a file listed beside a member resolving into it, in a new plan-gate-only pass
9 — resign-arrow-parse — parse the re-sign arrow on Edit bullets and refuse it on a non-member target
10 — resign-head-match — match each re-sign head to its member and refuse the arrow on an interface method
11 — caller-coverage — require every caller of a deleted or re-signed member to be covered by a target
12 — plan-stencil-and-rubric — teach Plan-Write member-glyph Edit targets, the arrow and caller coverage, and give Plan-Review two findings

## Shared Decisions

### Decision: this plan's own target shape

- **Decision:** Each Go file is named the same way on every card: either as a file path on every card that touches it (`internal/planparser/validate.go`, `shape.go`, `handle.go`, `parse.go`, `internal/planglyph/deleteorder.go`, the glyph-chain test files, every `doc.go`) or by its members on every card (the rest).
  A new symbol gets a `plan:` handle only when a later card targets or uses it; a helper only its own card uses rides its file's target.
- **Rationale:** This plan is validated and dispatched by the deployed lyx, which still runs cross-card containment and `handle-unreferenced` until this task lands; a file named on one card while another card names one of its members is refused there, which also rules out a later card naming a file a bound handle lives in.
- **Applies to:** all cards

### Decision: glyph-chain test layout

- **Decision:** The D1 fixture module lives at `internal/planglyph/testdata/glyphchain`, which repo scans skip as testdata.
  One table of shape rows feeds two files: `glyphchain_test.go` (untagged) runs the plan-gate legs, which read files and spawn no process, and `glyphchain_integration_test.go` (`integration`) runs the legs that need a real commit's delta.
  Rows for D2, D3, D4 and D5 land in these files in their decisions' own cards.
- **Rationale:** `PATTERN-test-speed` puts each test in the lowest tier its substrate needs; only the delta legs spawn git.
- **Applies to:** cards 1-11

### Decision: check IDs and where they run

- **Decision:** The new check IDs are `redundant-package-target` (planparser, every gate), `redundant-file-target` (planglyph, plan gates only), `resign-not-member` (planparser, every gate), `resign-head-mismatch` (planglyph, every gate), `resign-interface-method` (planglyph, plan gates only) and `caller-uncovered` (planglyph, plan gates only).
  "Plan gates only" means `planGatePass`, which `ValidateFormat` and `Validate` call and `ValidateDispatch` never does.
- **Rationale:** A check whose verdict depends on state the run itself changes (a bound handle's file, the tree's references, a member's kind) runs only where no card has run yet; a check that reads plan text alone runs everywhere.
- **Applies to:** cards 7-12

### Decision: quarry gap recorded, not fixed

- **Decision:** quarry indexes no struct fields, so a field glyph such as `shapes#Struct.Field` resolves `not_found`; the D1 struct-field row asserts that answer with a comment naming the gap, and the task summary names it for the operator.
- **Rationale:** Changes to quarry are out of scope; a defect rooted in quarry is recorded.
- **Applies to:** cards 1, 10

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
