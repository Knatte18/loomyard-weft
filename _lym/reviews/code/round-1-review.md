---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/raddleparser/proposal.go:60
    summary: The new pattern_clusters JSON key is never parsed by any test, so a tag typo or a broken unknown-key walk over the nested list would pass the suite.
    fix: In the raddleparser Parse test, add a row that parses a proposal JSON whose part carries pattern_clusters with sketch_id, entry and text and asserts the Part.PatternClusters values, and a row that refuses an unknown key inside a cluster object.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomshed/raddlefile_test.go:218
    summary: The mapped-change filing test never reads the project record, though Raddle-File writes the mapped record whole for the project and the spec says so.
    fix: In the MappedPatternChangeIsFiledUnderItsLiveEntry subtest, for the project shape also read f.deps.PartRecordPath(proposal.Project.Slug) and assert it holds the live entry's changes bullet and not PATTERN-a.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/decision_test.go:343
    summary: A withdrawal through a --from file has only refusal rows, so a successful --from withdrawal and its pattern_withdrawn envelope are unpinned.
    fix: In TestDecisionVerb_AppendsAndCommits, add a row whose input carries a --from file with one entry that has pattern_action withdraws and no pattern_text, and assert lastAdded.WithdrawnPattern and the pattern_withdrawn key.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/discussionparser/append_test.go:190
    summary: Two insertDecision withdraw branches and the fenced Withdraws line have no row - a PATTERN changes section placed before Decisions, a bullet on a last line with no newline, and a Withdraws line inside a fence.
    fix: In TestAppendDecision_WithdrawnPatternStrikesBullet add a row whose record puts the PATTERN changes section before ## Decisions and a row whose bullet is the record's final line with no trailing newline. In TestCheckAddedDecisionsKept add a row where the Withdraws line sits in a fenced block inside the added entry and still yields the dropped-bullet finding.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/raddlecontext_integration_test.go:43
    summary: raddleBoardContext's missing-PATTERN.md branch, where no id is live, has no test.
    fix: In TestRaddleBoardContext add a case that resolves a worktree without a PATTERN.md and asserts LivePattern("PATTERN-live") is false and the call returns no error.
---

### [LOW] pattern_clusters is never parsed by a test

**Location:** internal/raddleparser/proposal.go:60
**Issue:** Every test builds `PatternClusters` as a Go literal.
The JSON route a real proposal takes, `Parse` with its reflective unknown-key walk, is never run over the new key.
`pattern_clusters` is optional, so a mistyped tag would drop every mapping silently.

### [LOW] The project record's mapped bullet is not asserted

**Location:** internal/loomshed/raddlefile_test.go:218
**Issue:** The subtest runs the three shapes but reads only the record of `part-a`.
For the project shape, `writeRecords` writes the mapped Sketch record under `proposal.Project.Slug`, and the raddle proposal spec says the project record is covered.

### [LOW] A successful --from withdrawal is untested

**Location:** internal/loomcli/decision_test.go:343
**Issue:** The `--from` coverage for `withdraws` is two refusals.
The success path through `decisionEntries` and the `pattern_withdrawn` envelope is exercised only through the flag form.

### [LOW] Withdraw branches without a row

**Location:** internal/discussionparser/append_test.go:190
**Issue:** `insertDecision` splices differently when the PATTERN changes section precedes `## Decisions`, and guards `patternEnd < len(content)` for a bullet on an unterminated last line.
`withdrawnPatternIDs` skips fenced lines.
No row reaches any of the three.

### [LOW] No test for a worktree without PATTERN.md

**Location:** internal/loomcli/raddlecontext_integration_test.go:43
**Issue:** `raddleBoardContext` treats a missing `PATTERN.md` as holding no live entry.
Only the file-present case is tested.

## Rejected

- Missing nil check on `BoardContext.LivePattern` at `internal/raddleparser/check.go:220` (lens:code-correctness): the one production builder sets it, `RecheckBoard` never reaches the cluster check, and the code-quality rule trusts internal callers.
- Lens code-affected-set returned no findings, and the handler's own pass over the diff against the decision record found none: both halves of the board entry are made, no change lacks a licence, the docs named by the repository's own rules land in the diff, the commits carry no `Pattern-Departure:` trailer, and `_lym/landing/summary.md` is borne out by the diff.
