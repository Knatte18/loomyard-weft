---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:71
    summary: Decision 6's search procedure leaves a jointly conflicting pair proposed when some other fix also fails its solo run
    fix: In decision-record.md Decision 6, replace the sentence starting When no solo run does with one that covers the mixed case without an extra run, e.g. Every new blocking finding of the combined run that no solo run reproduces is a joint conflict, and the agent withholds the fixes that finding names by card and reports those defects together, the same way, whether or not some other fix failed its solo run.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:31
    summary: Decision 1 moves delete-before-reference out of the resolve-backed pass at the plan gates but names a new home only for deleted members, not for a package self glyph Delete
    fix: In decision-record.md Decision 1, after the sentence naming where the typed, partitioned form runs, add that a package self glyph Delete's import-path match (Decision 3) runs in the same planGatePass at the same entry points, over the same later Edit regions, and stays blocking.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:63
    summary: Decision 5's refusal list omits two refusals the acceptance row requires with a way forward, a Card Index name with no file on disk and the combination with --rework or --require-approved
    fix: In decision-record.md Decision 5's Bound and refusals sentence, add a card name the Card Index lists whose file is absent on disk, and --overlay combined with --rework or --require-approved, so each gets its refusal-spec.md row and test alongside the others.
---

The review judges the decision record and support log against the Discussion-Review rubric and this round's focus directive, checked against `internal/planglyph/planglyph.go`, `deleteorder.go`, `callercoverage.go`, `index.go`, `internal/loomcli/validate.go`, `internal/loomshed/gates.go`, `internal/websterengine/recordbatch.go` and `recoverbatch.go`.

Focus verdicts, one per directive entry:

- Overlay writes and reads (Decision 5): settled, not a defect.
  Decision 5 replaces `CanonicalizeHandles`' `RewriteRefs` write with an in-memory substitution and the post-rewrite `ParsePlan(planDir)` reload with a re-parse through the overlay reader, and Decision 6 makes the reviewer's baseline `--overlay` with `{}`, so no reviewer run writes.
  The acceptance row on a non-canonical plan handle covers the case.
- Done card at the plan gate (Decisions 1 and 4): settled, not a defect.
  `index.ValidateFormat` with done cards reaches `validateFormatAfter`, which runs `ValidateDispatch` and then `planGatePass` over the pending cards; Decision 1 names the typed form's entry points and drops the check from the shared resolve-backed pass at the plan gates, and an acceptance row covers a done card.
- `recover-batch` (Decision 4): settled, not a defect.
  Decision 4, Scope and the acceptance row each state that `ErrRecoveryDeleteReferenced` fires on blocking findings only and still fires on a blocking package-level reference.
- Overlay name rule (Decision 5): settled; `amendments.md` and orphan files are refused, and terminal stdin is in the acceptance row.
  The refusal list's divergence from the acceptance row is F3.
- Skipped directories (Decision 3): settled, not a defect.
  Decision 3 states the `testdata`, `vendor`, `.` and `_` skip with its bound, matching `candidateFiles`, and an acceptance row covers the `.go`-only import-path match.
- Decisions 6 and 7: Decision 7 names the file's self glyph when no member span holds the line, settled.
  Decision 6 handles a pure joint conflict but not a mixed one (F1).

### [LOW] Decision 6 misses a joint conflict alongside a solo failure

**Location:** `_lyx/discussion/decision-record.md:71`
**Issue:** The procedure withholds solo-failing fixes, and only "when no solo run does" withholds the fixes a combined-run finding names.
With fix A failing solo and fixes B and C failing only together, A is withheld and B and C are both proposed, and no run checks them together again.
That breaks Decision 6's own rule that no proposed fix's overlay run reports a blocking finding the baseline does not.
A combined-run finding no solo run reproduces is identifiable from the runs Decision 6 already makes, so the fix needs no extra run and leaves the bound unchanged.

### [LOW] Decision 1 does not place the package self glyph Delete check

**Location:** `_lyx/discussion/decision-record.md:31`
**Issue:** Today `LaterDeleteReferences`, called from `resolvePass`, handles both member and package self glyph Delete targets (`deleteorder.go`, `shapePackage`).
Decision 1 removes `delete-before-reference` from the resolve-backed pass at the plan gates and moves "each deleted member" into the typed computation, but `caller-uncovered`'s subjects are member glyphs only (`coverageSubjects` in `callercoverage.go`).
Decision 3 keeps the package import-path match without saying which plan-gate pass runs it, so a plan writer following Decision 1 literally could drop the check at the plan gates.

### [LOW] Decision 5's refusals and the acceptance row disagree

**Location:** `_lyx/discussion/decision-record.md:63`
**Issue:** Decision 5 binds each listed refusal to a way forward, a `refusal-spec.md` row and a test.
The acceptance row at line 118 also refuses "a name with no file on disk" and "a combination with `--rework` or `--require-approved`", each with its way forward.
Decision 5's name rule admits any card the on-disk Card Index lists, including one whose file is missing, which would let the overlay add a file and contradict Decision 5's "adds and removes no file".
The existing `--require-approved`/`--rework` exclusion in `internal/loomcli/validate.go` has no way forward and no `refusal-spec.md` row, so a plan writer working from Decision 5's list alone would not add one.
