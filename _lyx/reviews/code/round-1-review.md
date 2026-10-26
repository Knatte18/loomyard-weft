---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: lens:code-pattern-fit
    location: PATTERN.md:34
    summary: PATTERN-shed-stop-chain still says rule table, tackler, arbiter, then parent for every recipe, but the new row sends a blocked Webster stop to the arbiter and skips the tackler, and the diff leaves PATTERN.md unchanged
    fix: In PATTERN.md edit the PATTERN-shed-stop-chain entry so it names the exception, keeping the rest of the entry and its one-line shape, for example "A shed run's stop is settled by the rule table, tackler, arbiter, then parent, for every recipe; a blocked stop at a row the driver's arbiter seam marks as judged first (Webster's) goes rule table, arbiter, then parent; arbiter keys in shed.yaml with module overrides." Land it in the same branch as the code, as CLAUDE.md requires for a cross-cutting invariant. No Pattern-Departure trailer is needed once the entry says it.
  - id: F2
    severity: MEDIUM
    class: scope
    origin: lens:code-pattern-fit
    location: internal/websterengine/runlevel.go:740
    summary: The new refusal in Run for an unreadable decision record, with its way-forward text, and Run's pass-through of Geometry.DecisionRecordPath into the master prompt have no test, though PATTERN-refusal-way-forward requires a test to reach each webster refusal
    fix: Add rows to an existing Run test in internal/websterengine. One row sets Geometry.DecisionRecordPath to a record holding an Added after Discussion entry and asserts the master prompt the run writes carries that entry under the parent decisions section. One row sets it to a directory path and asserts Run returns an error naming the record path and the re-entry way forward (way forward, transient). Add no new test function if a Run test already builds that fixture.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/websterengine/template_test.go:839
    summary: The master renders none row asserts a stencil sentence, the blank lines around the marker and the next heading, which is an assertion on the stencil's static prose
    fix: In the row "master renders none for no parent decision" replace the contains string "never raise a question one of them answers.\n\nnone\n\n## A plan-drift" with an assertion that does not quote stencil prose, for example the heading "## A parent decision reaches the code through your forks" and "none" in inOrder with the next heading "## A plan-drift", or contains only the rendered marker line and notContains "{{.parent_decisions}}".
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/loomcli/drivewiring.go:155
    summary: The wiring that sets JudgesFirst for the Webster row only, with darn and batten leaving it unset, has no test, so marking another row, none, or batten too would pass every test
    fix: Add a row to an existing loomcli test that builds the driver deps through goDriverDepsFor, asserting Arbiter.JudgesFirst is non-nil and true for loomshed.NameWebster and false for another row name. If no loomcli test reaches goDriverDepsFor, extract nothing new and cover it through the nearest test that does build the deps.
---

### [BLOCKING] PATTERN-shed-stop-chain is left false for Webster

**Location:** PATTERN.md:34, with the new row at internal/sheddriver/driverules.go:234
**Issue:** The entry reads "rule table, tackler, arbiter, then parent, for every recipe".
The diff adds a row that sends a blocked stop at a `JudgesFirst`-marked row, which the loom wiring sets to Webster, to the arbiter with no tackler call.
The fasit licenses the behaviour, and the decision record's constraint admits the exception ("the order for every stop the new row does not mark"), but it declares no PATTERN change, and the diff and both commit messages leave the entry and any `Pattern-Departure:` trailer out.
CLAUDE.md puts a new cross-cutting invariant into PATTERN in the same commit as the code, and the repository's rules make a violation of a listed constraint BLOCKING.
The affected-set lens rated the same defect MEDIUM; the rubric's rule for listed constraints decides the severity.

### [MEDIUM] The Run refusal for an unreadable decision record is untested

**Location:** internal/websterengine/runlevel.go:740
**Issue:** `Run` now reads the record through `discussionparser.AddedDecisionTexts` and returns a refusal that names a way forward when the read fails.
`TestAddedDecisionTexts` covers a missing record and an empty path, but not a read error, and the prompt tests call `RenderMasterPrompt` directly, so no test reaches `Run`'s read, its pass-through of `Geometry.DecisionRecordPath` or the refusal.
The code-pattern-fit and code-tests lenses raised this separately; kept once.

### [LOW] A stencil prose assertion in the none row

**Location:** internal/websterengine/template_test.go:839
**Issue:** The row asserts the stencil's closing sentence, two blank lines, `none` and the next heading, which pins static prose and breaks on any rewording of the paragraph.
The order row's `inOrder` and `TestTemplates_FillRequiresEveryRequiredMarker` already pin the section heading and that the marker is filled.

### [LOW] The Webster-only marking is not pinned

**Location:** internal/loomcli/drivewiring.go:155
**Issue:** The driver tests use a fake `JudgesFirst`, so nothing fails if the wiring marks a different row, no row, or darn and batten as well.
The fasit's scope is the Webster row only.

## Focus verdicts

- Row order and nil seam: the new row sits after the hand-over and tackler-resume-returned rows and after the landing-halt, budget-exhausted, precondition-unmet and pause rows, and a nil `Arbiter` or nil `JudgesFirst` leaves `ArbiterFirst` false, so the tackler path is unchanged. Not a defect.
- Wiring: only the Webster row is marked, through the seam, and the rule table stays recipe-blind. Not a defect apart from F4.
- Parser: `AddedDecisionTexts` returns heading and body in record order up to the next heading, skips fenced headings, and shares `addedDecisionSpans` with `IdentityText`, so the discussion parser stays the sole parser of the record. Not a defect.
- Render: `parent_decisions` is a required marker filled from `Geometry.DecisionRecordPath`, renders `none` for no entries and for an empty path, and a read error refuses before any prompt is rendered. Not a defect apart from F2.
- Stencil: the stuck rungs name no message, the rule is stated once at the head of the ladder, and the waiting and decision rungs still message. Not a defect.
- Docs: the orch role stencil already says the driver hands a stop to the arbiter first, so it needs no edit; `internal/sheddriver/doc.go` and `internal/websterengine/doc.go` are updated. Not a defect.
- Tests: no new test duplicates an existing one; F3 and F4 are the gaps. No `Pattern-Departure:` trailer appears, and the summary's claims match the diff.

## Rejected

- Pause row ordering has no test row with `ArbiterFirst` set on a paused stop (code-tests, item 5): the lens called it minor, and the paused case returns in the table's earlier rows that existing rows already cover.
- A test for a read error of `AddedDecisionTexts` other than not-exist (code-tests, item 7): folded into F2, which reaches that error through `Run`.
