---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:51
    summary: Decision 5 rests on a false premise, since the check path writes card files to disk through handle canonicalization and then re-parses the plan from disk, so an overlay run both writes plan files and silently drops the overlay
    fix: In Decision 5, add a requirement and its bound for handle canonicalization under --overlay - state that resolvePass's CanonicalizeHandles step writes nothing in overlay mode (its substitutions are applied to the overlay reader's contents in memory, or the overlay run reports a non-canonical handle as a finding instead of rewriting), and that the post-canonicalization re-parse reads through the same overlay reader, never planparser.ParsePlan on disk. Add an acceptance row - an overlay that adds a non-canonical plan handle leaves every plan file byte-identical and its later checks see the overlaid card. In Decision 6, state that the reviewer's flag-less baseline run must also write nothing (or that the rubric runs only the overlay form, with {} as its baseline), since the reviewer may write only its review file.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:27
    summary: Decisions 1 and 4 leave open which form of delete-before-reference the plan gate runs once Webster has done cards, where ValidateFormatAfter runs ValidateDispatch's scan-only set and planGatePass's typed set over the same plan
    fix: In Decision 1, add a sentence naming which entry points run the typed, partitioned form - ValidateFormat, Validate, ValidateRework and ValidateFormatAfter over its pending cards - and state that the resolve-backed pass shared with ValidateDispatch no longer runs delete-before-reference at those plan gates, so ValidateFormatAfter reports each reference once, from the typed form. Add an acceptance row - with a done card in the run record, the plan gate reports a later-region method call once, blocking when the load answers, and never also an informational duplicate.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:45
    summary: Decision 4 settles that batch recording counts only blocking findings, but leaves recover-batch's delete-referenced refusal undecided, though it reaches the same check and will now see informational name-only findings
    fix: In Decision 4, state recover-batch's behaviour - its ErrRecoveryDeleteReferenced refusal fires on blocking delete-before-reference findings only, as batch recording's way forward does - and name it in Scope's internal/websterengine bullet. Extend the Webster acceptance row - recover-batch neither refuses on a name-only method match nor lets a blocking package-level reference through.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:40
    summary: Moving onto caller-uncovered's candidate walk stops searching later Edit regions in Go files under testdata, vendor and dot- or underscore-prefixed directories, a guard weakening Decision 3's bound does not name
    fix: In Decision 3, either state that a later card's Edit region in a .go file under a directory the go tool ignores (testdata, vendor, a name starting with . or _) is still searched, or add that skip to the Decision's bound with the reason it never breaks the package build.
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:57
    summary: Decision 5's name rule accepts any existing top-level .md file, so an overlay of amendments.md or an orphan .md file is accepted and has no effect, letting a reviewer believe a proposed change was validated
    fix: In Decision 5, narrow the accepted names to 00-overview.md and the card files the on-disk Card Index lists, and refuse any other name with its way forward. Add that case to the refusal acceptance row.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:104
    summary: Two Decision behaviours have no acceptance row - Decision 5's refusal of terminal stdin, and Decision 3's limit of a package self glyph Delete's import-path match to .go Edit regions
    fix: Add terminal stdin to the refusal list in the acceptance row at line 104, and add a row - a package self glyph Delete whose quoted import path appears in a later card's .md Edit target yields no finding, while the same import in a later .go Edit region stays blocking.
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:62
    summary: Decision 6's search for the offending fix is unspecified and its bound assumes a single offender, so a blocking finding that only two fixes produce together has no stated disposition
    fix: In Decision 6, state the procedure as - re-run with each fix overlaid alone; a fix whose solo run reports a new blocking finding is withheld and reported as a defect with the constraining check; when no solo run does, withhold the fixes the combined run's new findings name by card and report them together. Keep the bound of one combined run plus one solo run per proposed fix.
  - id: F8
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:68
    summary: Decision 7 does not say what the reviewer names when no function or method span encloses the cited line, such as a free-standing comment between declarations
    fix: In Decision 7, add - when lyx quarry resolve shows no member span holding the cited line, the finding names the file's self glyph instead of a neighbouring member.
---

### [BLOCKING] The overlay check path writes card files and re-reads the plan from disk

**Location:** `_lyx/discussion/decision-record.md:51`

**Issue:** Decision 5 says nothing is written to disk and that every read of plan content goes through an overlay-first reader, naming only the parse and the done-card comparison.
The check path the overlay runs does more than that.
`resolvePass` (`internal/planglyph/planglyph.go`) calls `CanonicalizeHandles`, which calls `planparser.RewriteRefs(planDir, subs)` (`internal/planglyph/handle.go`).
`RewriteRefs` (`internal/planparser/rewrite.go`) re-parses the plan with `ParsePlan(planDir)` from disk and writes changed card files back.
`resolvePass` then reloads with `planparser.ParsePlan(planDir)` and runs the status, create and delete-order checks on the reloaded plan.
Under an overlay that rewrite fires whenever the overlaid or on-disk plan holds a non-canonical handle, for instance a proposed fix that adds a `plan:` Create handle.
Two failures follow: the run writes plan files, breaking Decision 5's "nothing is written" and the Plan-Review reviewer's one-file write surface that Decision 5's own rationale depends on;
and the reload discards the overlay, so the later checks judge the disk plan and can pass a fix the overlaid plan would fail.
Decision 6's flag-less baseline run has the same write, which the reviewer's write surface also forbids.
The acceptance row on byte-identical files would fail on this path, and nothing in the record tells the plan writer to handle it.
`cardHeadingNumber` (`internal/planparser/validate.go`) re-reads card files from disk too; Decision 5's "every read" requirement covers it, so it is not raised separately.

### [MEDIUM] Which delete-before-reference form the plan gate runs after Webster has done cards

**Location:** `_lyx/discussion/decision-record.md:27`

**Issue:** `LaterDeleteReferences` runs today from `resolvePass`, which both the plan gates and `ValidateDispatch` call.
Decision 1 puts the typed, partitioned form at the plan gates, and Decision 4 keeps `ValidateDispatch` scan-only.
The plan gate's index entry `ValidateFormat` calls `validateFormatAfter`, which, once the run record holds a done card, runs `ValidateDispatch` and then `planGatePass` over the pending cards (`internal/planglyph/planglyph.go`, `internal/planglyph/index.go`).
On that path the plan gate would run both forms: a name-only method call yields an informational scan-only finding and a blocking typed one for the same line, and a package-level reference is reported blocking twice.
The record does not say that the resolve-backed pass stops running the check at the plan gates, and no acceptance row covers a plan gate with done cards.

### [MEDIUM] recover-batch's delete-referenced refusal is left undecided

**Location:** `_lyx/discussion/decision-record.md:45`

**Issue:** `laterDeleteReferenceReasons` (`internal/websterengine/recordbatch.go`) feeds two refusal surfaces: batch recording's `failCardNotDone`, and `recover-batch`'s `ErrRecoveryDeleteReferenced` refusal (`internal/websterengine/recoverbatch.go`).
Decision 4 and the Scope bullet state the blocking-only rule for batch recording alone.
Once name-only method matches become informational findings, whether `recover-batch` still refuses on them is a behaviour choice with its own way forward, not a call-site listing; leaving it open lets the plan writer fix one surface and leave recovery refusing on the same false positives #562 describes.

### [LOW] The candidate walk's skipped directories are an unbounded weakening

**Location:** `_lyx/discussion/decision-record.md:40`

**Issue:** Today's matcher reads every later Edit region's file directly.
`candidateFiles` (`internal/planglyph/callercoverage.go`) skips `testdata`, `vendor` and directories starting with `.` or `_`, so a shared computation drops those regions from `delete-before-reference`.
Decision 3 bounds only the comment, string-literal and non-Go drops; this one is neither stated nor bounded.

### [LOW] The overlay accepts .md files that are not plan files

**Location:** `_lyx/discussion/decision-record.md:57`

**Issue:** The plan directory may hold `amendments.md` (`planparser.AmendmentsFileName`) or an orphan `.md` file.
Each is an existing top-level `.md` file, so Decision 5 accepts it, yet `ParsePlanFrom` never reads it, so the overlay silently changes nothing and the reviewer reads the run as validating its proposal.

### [LOW] Two Decision behaviours lack acceptance rows

**Location:** `_lyx/discussion/decision-record.md:104`

**Issue:** Decision 5 refuses stdin that is a terminal, but the refusal row lists empty stdin only.
Decision 3 and Scope's Out bullet limit a package self glyph Delete's import-path match to `.go` Edit regions, and no row covers that change.

### [LOW] Decision 6's bisection procedure and bound

**Location:** `_lyx/discussion/decision-record.md:62`

**Issue:** "re-runs without fixes until it finds the offending one" does not say whether fixes are dropped one at a time or tried alone, and assumes one offender.
Two fixes that only together produce a blocking finding have no disposition, and the bound of one re-run per fix does not say which runs it counts.

### [LOW] No disposition when no span encloses the cited line

**Location:** `_lyx/discussion/decision-record.md:68`

**Issue:** The premise holds: `lyx quarry resolve` returns each symbol's `file`, `start` and `end`, and `start` includes the doc comment, so a doc-comment line lies inside its own function's span (checked by running it on `internal/planglyph#insideLaterEditRegion`).
For a line outside every member's span, a free-standing comment between declarations, the directive gives the reviewer nothing to name, which invites the near-function choice #563 reports.

## Verdicts on the focus entries

- Decisions 1, 2 and 4 agree on severity: a scan-only pass resolves a package-level reference by import path and reports it blocking, and a method reference by name only and reports it informational; each Decision names its bound. The disagreement is over which entry points run which form (F2) and recover-batch (F3).
- Decision 5's refusals and mutual exclusions bound the name and mode surface, and a done card or `00-overview.md` overlay reaches the done-card comparison and the disk listing as the record says; the gap is the canonicalization write and reload (F1) and non-plan `.md` names (F5).
- Decision 6 is actionable apart from the bisection procedure (F7); Decision 7's `lyx quarry resolve` premise is true (F8 covers the uncovered case).
- Every Scope item and Decision has an acceptance row except those in F6, and no row contradicts a Decision.
- Notes for the plan writer and Open risks carry no requirement missing from Decisions or Constraints; the Notes' code locations are a head start the stencil permits.
- Every requirement in the support log's Interview and Question ledger appears in the decision record.
