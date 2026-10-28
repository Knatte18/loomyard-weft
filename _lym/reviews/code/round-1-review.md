---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-gate-slot.md:28
    summary: The self-check parity pair for ValidateLoomDiscussion still lists only Validate plus CheckRelevant, though the diff adds the decision-id and citation checks
    fix: In pattern/PATTERN-gate-slot.md line 28, extend the parenthetical to name discussionparser.CheckDecisionIDs and the citation check (citationFindings via discussionparser.Citations) beside Validate and CheckRelevant.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: internal/loomshed/seamtest.go:130
    summary: coversSeamTest counts the test's file named under any group (Create, Edit, Extends included) as a cover, wider than D6 and spec check 55, which allow the file only under Uses, Delete, Rename or Move
    fix: In coversSeamTest, match the test file path only against the refs of Uses and the Delete, Rename and Move groups (strippedRefs(card) is every group); or, if the wider cover is intended, widen D6's sentence in contracts/specs/loom-plan-spec.md check 55 to say so.
---

### [LOW] Stale parity pair in PATTERN-gate-slot background

**Location:** pattern/PATTERN-gate-slot.md:28
**Issue:** The background file of an entry the diff touches describes `ValidateLoomDiscussion` as `Validate` plus `CheckRelevant`. The diff makes it also run `CheckDecisionIDs` and the citation check, so the line is wrong and CLAUDE.md's "Docs land in the same commit" is not met.

### [LOW] Seam-test cover by file is wider than the record

**Location:** internal/loomshed/seamtest.go:130
**Issue:** `coversSeamTest` calls `strippedRefs(card)` with no group types, so a Create, Edit or Extends file target naming the test's file covers the test. D6 and spec check 55 name only Uses, Delete, Rename and Move for the file route. The effect is a missed `seam-test-unlisted` finding in a narrow case.

## Rejected

- code-tests F1 (no `doc-go-unlisted` row at the D9 wiring sites): `docGoFindings` is called unconditionally inside `planContextFindings` and reads no builder-supplied field, so no builder or mode can drop it; the tier and seam checks, which need builder fields, are pinned at the wiring sites.
- code-tests F2 (no `--rework` or `--overlay` row for seam): both modes use the same `PlanContext` builder as the no-flag mode, which `parity_test.go` pins.
- code-tests F3 (no test for `loomCLI.wire` filling `Env.TierTags`): a coverage preference on a one-line assignment, not an unpinned behaviour the record asks for.
- code-tests F4 (Rename and ambiguous rows for D8 unconfirmed): `RenamingATestPasses`, `RenamingATestHelperPasses` and `EditingAnAmbiguousMemberFails` exist at `internal/loomshed/gates_test.go:1178-1190`.
- code-tests F5 (quarry outage at the gate level): the gate closure returns the error verbatim; the outage is pinned at `ValidateLoomDiscussion`.
