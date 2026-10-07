---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/planglyph/doc.go:24
    summary: the new code-index paragraph is wrapped at a fixed column mid-sentence and puts two sentences on one line, which PATTERN-comment-line-breaks forbids
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/verifygate.go:102
    summary: the new checkPublishVerify doc breaks a list at a comma that precedes no conjunction
  - id: F3
    severity: BLOCKING
    class: consistency
    location: internal/commentlint/doc.go:14
    summary: the finding-definition sentence breaks at a comma before nor, and the clause after it has no subject or verb of its own
  - id: F4
    severity: BLOCKING
    class: consistency
    location: internal/landingshed/publish_verify_test.go:136
    summary: the test doc breaks at a comma before and that, a second object clause of pins and not an independent clause
  - id: F5
    severity: BLOCKING
    class: design
    location: internal/shedrecipe/entries_gate_test.go:58
    summary: the new test TestResolveGateSpec_PlanGatesRequirePlanIndex neither calls t.Parallel nor names the global state that prevents it, against PATTERN-test-speed
  - id: F6
    severity: BLOCKING
    class: design
    location: internal/commentlint/lint_test.go:91
    summary: TestFindNewWrappedBreaks_FindingNamesFileLineAndText pins the Finding fields that TestLintCommentsCmd_OverARepository already asserts at the public surface, against PATTERN-test-economy
  - id: F7
    severity: MEDIUM
    class: design
    location: internal/impactset/guards.go:101
    summary: a marker in a file no integration-tier run compiles is reported with the file only, while card 8 and the round gate promise file and line
  - id: F8
    severity: MEDIUM
    class: design
    location: internal/commentlint/lint.go:107
    summary: isProseBlock skips a whole comment block when any line is a heading, list item or indented code, so a wrapped paragraph next to one is never linted, which is how F1 passed the gates
---

## Method

The range is `git merge-base main HEAD`..HEAD, since `_lyx/loom/rework/` holds no round record and `product.parent` is `main`.
I built `lyx` from this branch and ran `lyx loom lint-comments --base <merge-base>`: it reports no finding.
I also ran `go build ./...`, `go vet` under every tag, `go test ./...`, `go test -tags integration` over the packages the diff touches, and `go test -tags tmux ./...`: all pass.
The budget file matches an independent count of top-level tests per package, and every test added in a new file calls `t.Parallel`.
The lint passes, yet the checks below found comment-convention violations it cannot see, so the verdict rests on reading the diff.

## Settled focus questions

- Comment breaks in `internal/planindex/planindex.go`, `internal/planglyph/repo.go`, `internal/websterengine/beginbatch.go` and `runlevel.go`: not defects.
  Their moved and edited comments break only at sentence ends, and each break an edit carried over unchanged is exempt by the pattern.
  The same scan over every added comment line found F1 to F4 elsewhere.
- `t.Parallel` in the `commentlint`, `impactset`, `loomcli` and `cmd/lyx` scenario and guard tests: every one calls it.
  F5 is the one new test in the diff that does not.
- Wrapped prose above a `//lyx:guard` or `//testtiming:keep` line: still flagged, because a directive line ends its block.
  `doc.go` states it and the row "a directive line ends its block, so the prose above it is still checked" pins it.
  The related whole-block skip is F8.
- Coordinating conjunctions in `internal/commentlint/lint.go`: `and`, `but`, `or`, `nor`, `yet` and `so` pass after a comma, and the row "a comma before anything else is flagged" keeps introductory phrases flagged.
  No defect.
- `internal/loomshed/verifygate.go` and `impactset.Derive`: a git or directory read failure is a returned error, and only a `GuardScanError` or a lint finding reaches the writer.
  Each lint finding names file, line and the move.
  A marker in a `tmux` or `llm` file names no line, which is F7.
- `contracts/specs/refusal-spec.md` row for the Webster-Burler verify gate: it names failing tests, lint findings and misplaced markers, each with a way forward, and `TestVerifyGate_RoundScenario` reaches each of them.
  No defect.
- `cmd/lyx/testdata/test-budget.yaml`: every row equals its package's count.
- `internal/websterengine` with no code index: `BeginBatch` returns `errNoIndex` naming `Geometry.Index` before any mutation, and the row "a geometry with no code index names the missing wiring" reaches it.
- `cmd/lyx/treesitterlink_test.go` and `treesitterlink_integration_test.go`: the per-helper parse test is gone.
  `TestTreeSitterLinkFindings` covers the failing and stale cases that the real run never reaches, so no remaining test is redundant.

### [BLOCKING] F1: the new planglyph doc paragraph is fixed-column wrapped
**Location:** `internal/planglyph/doc.go:24`
**Issue:** The paragraph added about `internal/planindex` and `NewIndex` breaks mid-sentence after "which links no" and after "NewIndex returns", and ends the first sentence in the middle of a line.
`PATTERN-comment-line-breaks` requires one sentence per line and allows a break only at a semicolon or a comma before a conjunction.
The lint skipped the paragraph because the block it sits in holds a heading or list line (F8).
**Fix:** Rewrite the paragraph with one sentence per line.

### [BLOCKING] F2: a list broken at a bare comma in the checkPublishVerify doc
**Location:** `internal/landingshed/verifygate.go:102`
**Issue:** "It returns a non-empty Stuck reason naming `publish_verify` when the command fails or the tree is dirty," continues on the next line with "a non-nil error for an infrastructure fault".
That is a list item break, which the pattern names as a violation.
The lint let it through because the same word pair already ends a line in the unchanged `check` doc.
**Fix:** Split the paragraph into one sentence per line, so the reason, the error and the proceed case are separate sentences or one unbroken line.

### [BLOCKING] F3: nor clauses broken at a comma in the commentlint package doc
**Location:** `internal/commentlint/doc.go:14`
**Issue:** "A finding is a line at a checked break that ends neither a sentence (...)," continues with "nor at a semicolon, nor at a comma whose next line starts with a coordinating conjunction".
The clause after the comma has no subject or verb of its own, so the break is a compound predicate, which the pattern leaves to review.
**Fix:** Restate the definition as separate sentences, or keep it on one line.

### [BLOCKING] F4: object clause broken at a comma in a test doc
**Location:** `internal/landingshed/publish_verify_test.go:136`
**Issue:** The doc of `TestPublishVerify_PublishVerifyCommand` breaks after "as the base command," before "and that its failure ends Stuck".
The second clause is a further object of "pins", not an independent clause.
**Fix:** Write the two behaviors as two sentences, one per line.

### [BLOCKING] F5: the new shedrecipe test runs serially without a reason
**Location:** `internal/shedrecipe/entries_gate_test.go:58`
**Issue:** `PATTERN-test-speed` requires `t.Parallel` unless the test touches process-global state and a comment names it.
`TestResolveGateSpec_PlanGatesRequirePlanIndex` builds its Env from `t.TempDir()` and touches no env var, cwd or shared fixture, and no comment names a reason.
Its siblings are serial too, but they predate the pattern line and this test is new.
**Fix:** Call `t.Parallel` in the test and in its subtests.

### [BLOCKING] F6: a per-helper test duplicates the public surface's coverage
**Location:** `internal/commentlint/lint_test.go:91`
**Issue:** `TestFindNewWrappedBreaks_FindingNamesFileLineAndText` pins that a `Finding` carries the file, line and text.
`TestLintCommentsCmd_OverARepository` already asserts the exact finding `wrapped.go:3: // a comment that wraps in the` at the verb, and `TestLint_OverARepository` asserts file and line at `Lint`.
`PATTERN-test-economy` and the rubric call a per-helper test the public surface already covers a defect.
**Fix:** Delete the test, and lower the `internal/commentlint` row of `cmd/lyx/testdata/test-budget.yaml` by one so the row keeps equal to the count.

### [MEDIUM] F7: the tmux and llm marker error names no line
**Location:** `internal/impactset/guards.go:101`
**Issue:** Card 8 says a marker in a `tmux` or `llm` file is a returned error naming the file and line, and the round gate hands that message to the writer as its finding.
The message "marks a guard test in a file no integration-tier run compiles" names the file only, and `TestFindGuardTests` pins only `p_test.go` for both rows.
The writer cannot tell which of a file's tests carries the marker.
**Fix:** Name the line of the first marker in the error, and make both rows assert it.

### [MEDIUM] F8: a heading or list line exempts its whole comment block from the lint
**Location:** `internal/commentlint/lint.go:107`
**Issue:** `isProseBlock` rejects a block when any of its lines is indented code, a list item or a heading, so every paragraph of a package doc that holds one `# Heading` is never linted.
`PATTERN-comment-line-breaks` describes a lint that flags every new break that ends neither a sentence, a semicolon nor a comma before a conjunction, and names no block-level exemption.
F1 is a new paragraph in such a block that passed every gate.
**Fix:** Judge each paragraph of a block on its own, split at blank comment lines, and skip only a paragraph that holds code, a list item or a heading.
Add a row for a wrapped paragraph beside a heading paragraph, and state the paragraph rule in `doc.go`.
