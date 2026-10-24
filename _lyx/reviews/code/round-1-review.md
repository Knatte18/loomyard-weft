---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/discussionparser/validate_test.go:258
    summary: a parser fixture line still names the removed ID PATTERN-hub-suffix in a live test file
    fix: In internal/discussionparser/validate_test.go line 258, replace the fixture entry ID PATTERN-hub-suffix with an invented ID that is not a PATTERN entry, such as PATTERN-old-hub-rule, and keep the rest of the line unchanged so the row still exercises a changes action.
---

### [LOW] Removed ID PATTERN-hub-suffix remains in a live test fixture

**Location:** internal/discussionparser/validate_test.go:258
**Issue:** The record's acceptance criterion says no live file outside `docs/research/`, `docs/benchmarks/` and `testdata/` cites a removed PATTERN ID.
The `PATTERN changes section of every action` row still feeds the parser the line ``- `PATTERN-hub-suffix` — changes: `-LYXHUB` is the sole suffix.``, and `PATTERN-hub-suffix` is no longer an overview entry.
The string is inert parser input, so nothing breaks, but it is the one leftover of the citation sweep that a removed-ID search finds.

## Rejected

- The decision record names `planglyph` as the owner of the `glyph-conversion-chokepoint` rule, but the rule landed in `internal/planparser/doc.go` (its existing "Glyph Conversion Chokepoint Invariant" section), and the enforcing test's message in `cmd/lyx/constraintchokepoint_test.go` points there; the board entry says only "the owning package", and `planparser` holds the chokepoint code, so this is not a defect.
- `lyxcwd` and `agentname` have no `doc.go`; the hub-suffix and agent-name rules landed in the package doc comment of `lyxcwd.go` and `agentname.go`, where the package doc already lives; this honors the owning-package placement.
- `trimpath-safe-root` and test-tiers' "bar darn's tree-scoped targets" clauses are absent from the overview lines; the declared texts omit them and both survive in the `pattern/PATTERN-gate-slot.md` and `pattern/PATTERN-test-speed.md` background files, so no rule vanishes.
- Remaining citations of removed IDs sit in `docs/research/` and `docs/benchmarks/` reports and are out of scope by the record.
