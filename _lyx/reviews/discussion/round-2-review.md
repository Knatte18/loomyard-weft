---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:71
    summary: D6's rationale restates D2's condition more narrowly, as reuse of existing setup only, and drops D2's calls to existing test helpers or fakes
    fix: In D6's Rationale line, replace "D2 replaces it for a card whose new tests reuse existing setup, and a card without one gives the fork no placement signal (Open risks)" with "D2 replaces it for a card that meets D2's condition, and a card that does not gives the fork no placement signal (Open risks)", so D6 points at D2's condition instead of restating it.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:77
    summary: Constraints cite a loom-plan-spec.md validation-table convention that the spec never states, and attribute the test-budget rule to PATTERN-test-economy, which does not carry it
    fix: "Replace the Constraints bullet at line 77 with: CLAUDE.md's docs-land-in-the-same-commit rule: the `edit-test-file` row in `loom-plan-spec.md`'s validation table, with D5's way forward, lands in the same commit as the check. In the bullet at line 78, keep the `PATTERN-test-economy` clause about extending an existing table, and attribute the clause about `cmd/lyx/testdata/test-budget.yaml` to CLAUDE.md's Test budget rule instead of to `PATTERN-test-economy`."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:95
    summary: Scope and D4 prescribe rewriting the Glyph.UnitPath sentence in loom-plan-spec.md and its twin in internal/planparser/doc.go, but no acceptance criterion covers that rewrite
    fix: "Under Acceptance criteria, append to the `loom-plan-spec.md` criterion at line 95, or add a bullet after it: neither `loom-plan-spec.md`'s card path resolution section nor `internal/planparser/doc.go` limits `Glyph.UnitPath` to the disk-existence checks, and each names it as the sole glyph-to-path call without listing its readers."
  - id: F4
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:56
    summary: D4's rewrite drops the two-disk-existence-checks limit, leaving the spec sentence's trailing clause, a member glyph is skipped by both, with no antecedent
    fix: "In D4, after \"without listing its readers\", add: and the sentence's member-glyph skip is stated of the disk-existence checks by name rather than of every `Glyph.UnitPath` reader."
---

### [LOW] D6 restates D2's condition narrower than D2 states it

**Location:** `_lyx/discussion/decision-record.md:71`

**Issue:** D2's rule fires for a card whose new tests "reuse an existing test's setup, or call existing test helpers or fakes".
D6's rationale says D2 replaces the plain path "for a card whose new tests reuse existing setup", dropping the helper-or-fake half.
A plan writer reading D6 alone could take the `**Uses:**` rule as narrower than D2 makes it.
The Open risks bullet ties back to D2 correctly; only D6's paraphrase drifts.

### [LOW] Constraints cite a convention the spec does not state, and misattribute the test budget

**Location:** `_lyx/discussion/decision-record.md:77`

**Issue:** The focus asks whether Constraints cite a rule that actually governs the `edit-test-file` check.
`loom-plan-spec.md` states no validation-table convention requiring a row's way forward or same-commit landing; many rows name a way forward, but as practice, not as a stated rule.
The rule that does require "way forward, row in the same commit, a test reaches it", `PATTERN-refusal-way-forward`, binds refusals in `contracts/specs/refusal-spec.md`, and that spec's loom section says the `validate-*` verbs' findings envelopes are verdicts, not refusals, so it does not govern this check either.
The same-commit requirement is governed by CLAUDE.md's "Docs land in the same commit" rule.
The way forward and the test are already this task's own requirements, in D5 and Acceptance criteria.
Separately, `PATTERN-test-economy` and its background file never mention `cmd/lyx/testdata/test-budget.yaml`; that rule is CLAUDE.md's "Test budget".
The constraint content is right, but the citations would send a plan writer to sources that do not hold it.

### [LOW] No acceptance criterion for the `Glyph.UnitPath` sentence rewrite

**Location:** `_lyx/discussion/decision-record.md:95`

**Issue:** Scope's `loom-plan-spec.md` bullet lists the `Glyph.UnitPath` sentence rewrite, and D4 extends it to `internal/planparser/doc.go`'s twin.
The criteria cover the spec's new row, its preamble naming and its examples, and the stencil's rule changes, but nothing covers the `UnitPath` sentence in either file.
Every other Scope item has a matching criterion, so this item is the one a reviewer of the built work has nothing to check against.

### [NIT] D4's rewrite orphans the spec sentence's member-glyph clause

**Location:** `_lyx/discussion/decision-record.md:56`

**Issue:** `contracts/specs/loom-plan-spec.md:171` reads "`Glyph.UnitPath` (glyph -> path, used only by the two disk-existence checks below on a **self** glyph; a member glyph is skipped by both, ...)".
D4 drops the "used only by the two disk-existence checks" limit, and "skipped by both" then refers to nothing.
The member skip is still true of the disk-existence checks (`diskPathForRef` in `internal/planparser/shape.go` skips a member glyph), so the clause should survive, attached to those checks.

## Focus verdicts

- **D4 and the `UnitPath` sentence.**
  The claim that the "used only by the disk-existence checks" limit is already false holds: `internal/planparser` already calls `Glyph.UnitPath` in `reviewclass.go`, `targetdirs.go`, `redundancy.go` and `validate.go`'s `resignTargetShape`, beside `shape.go`'s `diskPathForRef`.
  `internal/planparser/doc.go` carries the limit at its Path ownership paragraph ("used only by validate.go's disk-existence checks"), while its chokepoint paragraph already says "the only glyph->path call" without a reader list, so the twin D4 names exists.
  The prescribed wording, sole glyph-to-path call without listing readers, names no list that can go stale.
  Not a defect, apart from the orphaned clause in F4.
- **D3's exemptions.**
  Each states what it lets through and what bounds it.
  The ignored-segment exemption lets through a `_test.go` path under `testdata`, `vendor`, or a `.`- or `_`-prefixed segment, bounded to those segments by the go tool never building such a file as a test of its package.
  The done-card skip lets through a done card's listed test file at the format gate only, bounded by `done-card-edited`, which flags a done card whose file changed since its batch began, and the rework gate applies no skip.
  The `language: none` skip lets such a plan list any `_test.go` path, bounded by that plan forgoing every glyph check.
  Not a defect.
- **Open risks and D6.**
  The one open risk, a fork placing a new test in a new file, is runtime fork behaviour the code cannot answer.
  D6's `ScopeGuard` claim holds against `internal/planglyph/scope.go`: `targetsCover` maps a handle through `planparser.RefScope`, which sends a glyph-derived unit through `goPackageDir`, so a `<dir>_test` handle covers a symbol-less change in `<dir>`.
  D6's restatement of D2's condition is F1.
- **Acceptance criteria and Constraints.**
  Every Scope item has a criterion except the `UnitPath` sentence, F3.
  The Constraints citation is F2.
