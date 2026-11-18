---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/shellguard/rows_test.go:184
    summary: The retained contained test lacks the required keep marker.
    fix: Add //testtiming:keep directly above TestRows_SteersOpenWithWholeCommandRefusal, with a reason stating that it pins the prefix and nonempty refused part across every row and engine and excludes the prefix from notices, unlike TestCheckTexts and TestRows_EngineTexts. Keep the test and its assertions.
    origin: lens:code-tests
  - id: F2
    severity: LOW
    class: consistency
    location: internal/shellguard/rows.go:419
    summary: The new prefix leaves three comments describing the former steer opening.
    fix: In internal/shellguard/rows.go, change the comments on pythonRoutes, shellEditRefusal and headlessClaudeRefusal to describe the row-specific refusal text following the shared whole-command clause, rather than claiming each opens every steer.
    origin: lens:code-affected-set
---

### [BLOCKING] Mark the distinct refusal assertions as kept

**Location:** internal/shellguard/rows_test.go:184

**Issue:** The current redundancy report marks this test contained by `TestCheckTexts` and `TestRows_EngineTexts`.
Their assertions do not replace it.
The former checks the prefix only for Codex's Python row at line 54;
the latter checks engine wording and fallback at lines 124–141.
The new test checks the prefix and nonempty remainder for every row on both engines at lines 189–192.
It also excludes the clause from every notice at lines 194–195.
`PATTERN-test-speed.md:271` requires a keep marker for a retained contained candidate.
The round's rubric explicitly requires this disposition after comparing assertions.
The severity follows the directive that a listed constraint violation is BLOCKING.

### [LOW] Update the comments describing row-specific openings

**Location:** internal/shellguard/rows.go:419

**Issue:** Prefixing all steers makes three existing opening claims stale.
The comments on `pythonRoutes` at line 81, `shellEditRefusal` at line 93,
and `headlessClaudeRefusal` at line 181 say their text opens every corresponding steer.
The changed return now places the shared clause before each of those fragments.
These comments describe the fragments' position incorrectly after this change.

## Review evidence

Reviewed `71d6ee1a044d76998b1a3ff37a5eaec9113b83d5..HEAD` against task T7 and its decision record.
Every shared-table steer receives the clause, including nonempty engine overrides.
Empty overrides retain their fallback behavior.
The row text still identifies the refused command category; matching, switches and notices are unchanged.
The constructors allocate fresh maps, so repeated calls do not accumulate prefixes.

Removing only the new clause from the settings fixture yields the base fixture exactly after JSON normalization.
The focus's shortened fixture path was resolved to `internal/shuttleengine/claudeengine/testdata/settings-golden.json`.
The commit messages and landing summary match the diff; there are no departure trailers.

`git diff --check` passed.
The focused tier 0 and tier 1 gate could not acquire its slot because the sandbox denies writes under
`/home/knatte/Code/loomyard-LYMHUB/_bolt/.lym/gate/`.
No tests ran in that attempt; this environmental limit is not a finding about the diff.
