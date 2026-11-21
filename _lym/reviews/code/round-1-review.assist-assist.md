---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: contracts/specs/loom-discussion-spec.md:124
    summary: The unconditional bug-record rule also governs Go-authored Darn records with empty required content.
    fix: Limit the opening sentence of the bug-entry rule to records authored by Discussion-Write, preserving the three requirements and the parent-only waiver rule.
    origin: lens:code-affected-set
---

### [MEDIUM] Limit the bug-record rule to Discussion-authored records

**Location:** contracts/specs/loom-discussion-spec.md:124

**Issue:** The rule applies to any record for a bug-labelled entry.
`internal/loomshed/recordwrite.go:46` declares the sections that its record generator leaves empty.
`BoardRecord` writes the board body in Goal and leaves Constraints and Acceptance criteria empty.
The producer uses that record at `internal/loomshed/recordwrite.go:169`.
`contracts/stencils/darn/darn-template-write.md:33` forbids its writer from editing that record,
and the next line points to this spec as authoritative.
An ordinary Darn bug record therefore violates the new contract without an authorized writer to satisfy it.
The requested change concerns Discussion-Write.

The remaining focus checks found no defect.
Step 1's board response includes labels, and the added stencil paragraph preserves the existing rules.
The spec retains the parent-recorded waiver and the requirement to reach code already present on the merge base.
The rubric covers the three requested subjects and points to the authoritative spec.
The task requests a process enhancement, so the missing Root-Cause trailer is not a finding.
No executable code or test changed; no test run was needed for static prose.
