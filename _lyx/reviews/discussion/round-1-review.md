---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 and Scope (loom-plan-spec.md bullet)
    summary: D4 and Scope rest on the false premise that Glyph.UnitPath has two readers, so the prescribed spec edit adding a third reader writes a sentence that is still false
    fix: >-
      In decision-record.md D4, replace the clause "the spec sentence that limits `UnitPath` to the two disk-existence checks gains this third reader" with "the spec sentence in `loom-plan-spec.md`'s card path resolution section, and the matching sentence in `internal/planparser/doc.go`, drop their used-only-by-the-two-disk-existence-checks limit, which is already false, and name `Glyph.UnitPath` as the sole glyph-to-path call without listing its readers". In Scope's `loom-plan-spec.md` bullet, replace "the shape-classifier sentence that limits `Glyph.UnitPath` to the two disk-existence checks (D4)" with "the sentence that limits `Glyph.UnitPath` to the two disk-existence checks, rewritten as D4 says".
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints (second bullet) and D5 rationale
    summary: The record binds the plan check through PATTERN-refusal-way-forward, but that pattern's rows live in refusal-spec.md, whose loom section says validate verbs' findings are not refusals and have no row
    fix: >-
      In Constraints, replace the `PATTERN-refusal-way-forward` bullet with "`loom-plan-spec.md`'s validation-table convention: the `edit-test-file` row names its way forward and lands in the same commit as the check, and a test reaches the finding". In D5's rationale, replace "`PATTERN-refusal-way-forward` asks every refusal to name its way forward;" with "every plan check's row names its way forward;".
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (language none sentence)
    summary: D3's skip under language none states neither why nor what it lets through, which the attack-surface rule asks of every exemption
    fix: >-
      In D3, after "It is skipped under `language: none`", add "because such a plan has no glyph alphabet, so neither a self glyph for the check to read nor D5's member-glyph and handle way forward exists; the skip lets a `language: none` plan list any `_test.go` path under `**Edit:**`, and is bounded by that plan forgoing every glyph check".
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D6 rationale
    summary: D6 says D2 replaces the whole-file read, but D2 applies only to a card whose new tests reuse existing setup, which Open risks' first bullet already concedes
    fix: >-
      In D6, replace "Rationale: nothing in the fork's path depended on the plain test-file path beyond a whole-file read, which D2 replaces." with "Rationale: nothing in the fork's path depended on the plain test-file path beyond a whole-file read; D2 replaces it for a card whose new tests reuse existing setup, and a card without one gives the fork no placement signal (Open risks)."
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md Open risks (second bullet) and D6; support-log.md Question ledger Q7
    summary: The open risk on a plan handle's external-test unit mapping is answered by the code, since planparser.RefScope, which ScopeGuard's file coverage calls, takes a handle's glyph unit through goPackageDir
    fix: >-
      In decision-record.md, delete Open risks' second bullet; in D6, replace "a test file edited beside an internal-test handle is no deviation; for an external-test handle see Open risks." with "a test file edited beside a handle is no deviation, for an external-test handle too, since a handle's `<dir>_test` unit maps to its package directory." In support-log.md, change Q7 to "(resolved): does `ScopeGuard` map a handle's `<dir>_test` unit to its directory? Yes; its coverage maps a handle's glyph unit through the external-test unit mapping (D6)."
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria (stencil and spec bullets)
    summary: Two Scope items have no criterion, the check's name in the spec's out-of-entry-point preamble and the rewrite of the stencil's Create-test-file-only-for-an-uncovered-surface rule
    fix: >-
      In Acceptance criteria, extend the spec bullet to "`loom-plan-spec.md` carries the `edit-test-file` row with its way forward, names the check in the validation-check preamble as one the plan index appends at format and rework validation, and none of its examples lists a `_test.go` file under `**Edit:**`." Extend the stencil bullet with "and its rule on when a `**Create:**` test file appears no longer presumes a test file listed under `**Edit:**`".
  - id: F7
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D3 (testdata sentence) and Acceptance criteria (third bullet)
    summary: The testdata exemption is narrower than the go tool's ignored-directory set loom-plan-spec.md already names (testdata, vendor, and names starting with a dot or underscore), though the fixture-not-a-test reason covers all of them
    fix: >-
      In D3, replace "a path with a `testdata` segment is exempt, since such a file is fixture data, not a test" with "a path with a segment the go tool ignores (`testdata`, `vendor`, or a name starting with `.` or `_`) is exempt, since the go tool never builds such a file as a test of its package". In the third Acceptance criterion, replace "for a `_test.go` path with a `testdata` segment" with "for a `_test.go` path with a segment the go tool ignores".
---

### [MEDIUM] UnitPath reader count rests on a false premise

**Location:** `_lyx/discussion/decision-record.md` D4, last sentence of its third paragraph, and Scope's `loom-plan-spec.md` bullet.

**Issue:** D4 says the spec sentence "that limits `UnitPath` to the two disk-existence checks gains this third reader".
That sentence, at `contracts/specs/loom-plan-spec.md:171`, is already false, and so is its twin at `internal/planparser/doc.go:73`: besides `diskPathForRef` in `shape.go`, `Glyph.UnitPath` is read in `reviewclass.go:45`, `redundancy.go:92,121,129` and `targetdirs.go:224`.
Following D4 literally, the plan writer writes a spec sentence that names three readers and is still wrong.
This is not a call-site enumeration demand: the fix removes the enumeration rather than completing it, which `scribe:prose`'s no-enumerated-consumer rule also asks for.

### [LOW] PATTERN-refusal-way-forward is the wrong binding for a plan check

**Location:** `_lyx/discussion/decision-record.md` Constraints, second bullet, and D5's rationale.

**Issue:** `PATTERN-refusal-way-forward` binds refusals with a row in `contracts/specs/refusal-spec.md`.
That file's loom section (line 229) says the `validate-*` verbs' findings envelopes are verdicts, not refusals, and have no row there.
The record keeps the pattern's name but moves its row to `loom-plan-spec.md`, so the constraint cites a rule that does not govern the check.
The concrete obligations it carries (way forward in the row, same commit, a test reaches it) are right and stay; only the citation changes.

### [LOW] language none skip has no stated bound

**Location:** `_lyx/discussion/decision-record.md` D3, third sentence.

**Issue:** The rubric's attack-surface rule asks every exemption to say what it lets through and what bounds it.
D3 states the `testdata` exemption's reason and the done-card skip's bound, but the `language: none` skip has neither.
Under `language: none` no path is canonicalized to a self glyph, so the check could not fire anyway, and D5's way forward does not exist; the record should say so.

### [LOW] D6 overstates what D2 replaces

**Location:** `_lyx/discussion/decision-record.md` D6, rationale.

**Issue:** D6 says the whole-file read is replaced by D2, but D2 binds only a card "whose new tests reuse an existing test's setup, or call existing test helpers or fakes".
A card adding a self-contained test gives the fork no placement signal, which Open risks' first bullet states.
This settles the focus question: D6's claim that nothing in the fork's path depends on the plain path holds (`KickstartPack` already leaves plain paths and self glyphs out of the pack, `internal/planglyph/pack.go:26`), but "which D2 replaces" disagrees with the first Open risk.

### [LOW] The external-test handle open risk is already answered

**Location:** `_lyx/discussion/decision-record.md` Open risks, second bullet; D6; `_lyx/discussion/support-log.md` Q7.

**Issue:** `ScopeGuard`'s file coverage calls `planparser.RefScope` (`internal/planglyph/scope.go`, `targetsCover`).
`RefScope` (`internal/planparser/targetdirs.go:93`) strips the `plan:` prefix in `refUnit`, parses the body as a glyph, and passes the glyph unit through `goPackageDir`, its doc saying so for a `plan:` handle explicitly.
So a handle on `<dir>_test` covers its package directory, and the risk, D6's hedge and Q7 can close.

### [LOW] Two Scope items lack a criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, fifth and sixth bullets.

**Issue:** Scope's spec bullet includes naming the check in the validation-check preamble (the out-of-entry-point list at `loom-plan-spec.md:368-371`), and Scope's stencil bullet includes rewriting the rule that a `**Create:**` test file appears only "for a surface no existing test file covers" (`loom-template-plan.md:150`).
Neither has a matching criterion, so a plan can satisfy every criterion and skip both.
The UnitPath sentence also lacks one; F1's rewrite covers it once the spec bullet's criterion is read against D4.

### [NIT] testdata exemption narrower than the go tool's ignore set

**Location:** `_lyx/discussion/decision-record.md` D3, and the third Acceptance criterion.

**Issue:** D3's reason, that a `testdata` file is fixture data and not a test, holds equally for `vendor` and `.`- or `_`-prefixed directories, the set `loom-plan-spec.md:443` already names for `caller-uncovered`.
No tracked `_test.go` file sits under those today, hence NIT.

## Focus verdicts

- D3 and D4, dispatch: not a defect.
  `planglyph.ValidateDispatch` calls `planparser.ValidateFormat` through `dispatchFindings` (`internal/planglyph/planglyph.go:147`), and the index's `ValidateDispatch` appends only `CheckFabricVocabularyName` (`internal/planglyph/index.go:62-70`).
  A check appended in the index's `ValidateFormat` and `ValidateRework` never reaches dispatch.
  `lyx loom validate-plan` without flags goes through `loomshed.ValidatePlan`, which calls the index's `ValidateFormat`, so the first criterion holds.
  `--require-approved` and `lyx webster validate` go through the index's `Validate`, which appends nothing, the same as `card-fabric-reference`.
- D3, attack surface: the `testdata` exemption cannot slip a real test past the check, since the go tool never builds a file under a `testdata` segment as a test of its package; the one tracked case is the `internal/planglyph/testdata/glyphchain` fixture module, which is fixture data.
  The done-card and rework bounds are stated; the `language: none` bound is not (F3).
- D1 and D5: the premise holds.
  `checkHandleMalformed` refuses a `.go` file unit on a handle under a glyph-enabled language whenever a non-Rename claim reads its unit (`internal/planparser/validate.go:639`, `fileUnitRuleApplies`), and `internal/planglyph/glyphchain_test.go:205` binds `plan:shapes_test#draft`.
- D2 and D6: see F4 and F5.
- Scope, criteria and Constraints: see F2 and F6. The test-budget constraint is consistent with Scope: `test-budget.yaml` is a non-Go path, which stays legal under `**Edit:**`.
