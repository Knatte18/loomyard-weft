---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/09-resume-circling.md:26
    summary: "Card 9 adds a resume refusal for a recipe or decision read error with no refusal-spec row and no test, and its way forward (re-run the verb) cannot fix a malformed decision file"
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-carry-over-wiring.md:3
    summary: "Cards 3 and 7 add the Env fields CarryOver and RowReviewModels but leave battencli's TestWire_EverySeamFilled list and envkit nilLegal untouched, so that test fails once either card lands"
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/09-resume-circling.md:3
    summary: "Card 9 leaves two way-forward sites naming lyx loom start after a circling decision, the loom root Long text in internal/loomcli/cli.go and the circling paragraph of docs/overview.md"
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/01-carry-over-write.md:39
    summary: "Card 1's result check verifies only the H2 sequence, the segment's pair and other segments' entries, not that every line outside the replaced span equals the prior file as the added D2 bound decision requires"
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/06-review-budget-three.md:16
    summary: "Card 6 changes the escalation test's own small budget constant from 2 to 3 and claims this pins the template budget end to end, but that test sets the budget itself and never reads the template"
---

### [BLOCKING] Card 9's new resume read-error refusal has no spec row, no test, and a way forward that cannot work

**Location:** `_lyx/plan/09-resume-circling.md:26` (refusal text), `:34` (spec bullet), `:44-46` (tests).

**Issue:**
Card 9 says that for an awaiting run "a recipe or decision read error refuses with the error and a re-run of the verb".
This is a new refusal in `loomcli`, a module bound by `PATTERN-refusal-way-forward`.
The card's `refusal-spec.md` bullet names only two rows: the `resume: awaiting` row split in two, and the changed `circling: decision already recorded` row.
No row covers the read-error refusal.
The card's test list (`resume_test.go`) has rows for woken, `driver_not_parked`, no decision and PR-Gate, and none reaches the read-error path.
The decision record's Constraint requires "spec rows and a test reaching each refusal in loom".

The way forward is also wrong for one of the two cases.
`PendingCirclingDecision` returns an error for a malformed decision file, which does not heal on a re-run.
The refusal-spec row for the same condition on the Bouncer side (`circling decision unreadable`) says to fix or delete the named decision file.
The recipe-lookup case already has the row `circling: recipe lookup failed` (rebuild or reinstall lyx), a different way forward again.
So "a re-run of the verb" fits neither case.

**Fix:**
Split the read error into two refusal rows in `refusal-spec.md`'s loom section, for example `resume: circling decision unreadable` (fix or delete the named decision file, then re-run `lyx loom resume`) and `resume: recipe lookup failed` (the lyx build is broken, rebuild or reinstall).
Add a `resume_test.go` row for each, reached through a malformed decision file and a failing recipe lookup.
State both rows' way forward in the card's Intent.

### [MEDIUM] Cards 3 and 7 break battencli's every-seam-filled test

**Location:** `_lyx/plan/03-carry-over-wiring.md:3` and `_lyx/plan/07-segment-models.md:3` (edit lists); `internal/battencli/wire_seams_test.go`, `internal/testkit/envkit/envkit.go` (`nilLegal`).

**Issue:**
`envkit.NilSeams` reports every nil func and map field of `shedrecipe.Env`.
`battencli`'s `TestWire_EverySeamFilled` fails for any such field that is neither filled by batten's `wire` nor listed in its `intentionallyNil`, and `SegmentBounces` is the precedent for a field with a legal nil, held in `envkit`'s `nilLegal`.
Card 3 adds `Env.CarryOver` (a func) and card 7 adds `Env.RowReviewModels` (a map).
Batten fills neither, and neither card lists `internal/battencli/wire_seams_test.go` or an `nilLegal` entry.
`go test ./internal/battencli` therefore fails at each card's commit, and `loomcli`'s own copy of the test is covered only because loom fills both.

**Fix:**
Add `CarryOver` (loom-only, batten has no review segment) to `battencli`'s `intentionallyNil`, and `RowReviewModels` either there or to `nilLegal` beside `SegmentBounces`, since an empty map is the documented fallback.
Name the file in the owning card's Edit list.

### [MEDIUM] Card 9 leaves two resume wordings pointing at `lyx loom start`

**Location:** `_lyx/plan/09-resume-circling.md:3` (edit list); `internal/loomcli/cli.go:390-392`; `docs/overview.md:411-413`.

**Issue:**
D6 says every way forward over an awaiting Bouncer row names `lyx loom resume`, and the Constraints require docs to land with the behavior.
Card 9 moves the circling verbs' `resume` field, their `Long` texts, the escalation brief, `escalationReason` and two spec rows.
Two statements of the same step remain unlisted:
the loom root command's `Long` text in `internal/loomcli/cli.go` ("and `lyx loom start` then resumes the run"), and the `docs/overview.md` paragraph on review-segment escalation, whose sentences say the fork "resumes the run with `lyx loom start`" and that a batten-driven run "is resumed the same way with `lyx loom start`".
After card 9 both describe a step that no longer matches the verbs' output.

**Fix:**
Add `internal/loomcli/cli.go` and `docs/overview.md` to card 9's Edit list and name the two passages in its Intent.

### [MEDIUM] Card 1's result check is weaker than the bound the added D2 decision sets

**Location:** `_lyx/plan/01-carry-over-write.md:39`.

**Issue:**
The decision record's "D2 bound verified on the result" requires the write to re-read the new file and require that its H2 heading sequence equals the prior file's, that each segment has at most one well-formed marker pair, and that every line outside the replaced span equals the prior file's.
Card 1 checks the heading sequence and the segment's pair, but replaces the third condition with "every other segment's entry byte-identical to the prior file's".
That covers other segments' entries only.
Ordinary prose and the section's other lines, which D2's Bound and its acceptance criterion ("every byte outside the entry is unchanged") say the write must keep, are never compared on the produced bytes.
A splice bug that damaged them would pass the card's check.

**Fix:**
State the third condition as the added decision words it: every line outside the replaced span (the entry's lines, or the appended or removed lines) in the new file equals the corresponding line of the prior file.
Name the append and removal cases, and add a `carryover_test.go` row only if an existing row cannot assert it.

### [LOW] Card 6's escalation-test edit does not pin what it says it pins

**Location:** `_lyx/plan/06-review-budget-three.md:16` (and `internal/loomrecipe/escalation_test.go`, `budgetEscalationBudget`).

**Issue:**
The card says changing `escalation_test.go`'s budget to 3 pins "the template's budget end to end".
That test sets `env.ReviewMaxBounces = budgetEscalationBudget` itself and passes the same constant to `Routing`; it never reads `template.yaml`.
The template value is pinned by `config_test.go`, which the card already edits.
The constant is documented as "the small review budget the escalation tests run with", and the card does not say how the file's round-count assertions move with it.
D3 licenses test-kit defaults that mirror the template, and this constant mirrors nothing.

**Fix:**
Drop the `escalation_test.go` edit, or keep it only if the card states that it exercises a CONTINUE at round 3 escalating with cause `budget` (the D3 acceptance criterion) and names the assertions that change, without claiming it pins the template.

## Focus verdicts

- D2 (cards 1 and 2): carried, with the gap in F4 (third result-check condition). The five malformed marker sets, marker recognition, one-line rendering and the skipped-review no-op are all carried and none widens scope.
- D1 and D7 (card 5): carried, including the parse-error line and its fallback, the round variants, the stencil rewording and the PATTERN amendment; the CIRCLING bullet and checkpoint are left unchanged. Card 5's `Uses` of `bouncercarryover.go` names no symbol it reads, which is harmless since Webster runs cards in number order.
- D4, D5 and D6 (cards 6 to 9): carried, with the gaps in F1, F3 and F5; the `PATTERN-shuttle-provider-seam` and `PATTERN-config-strictness` constraints are carried.
- Attack surface: every new edge states a bound (cards 2, 3, 5, 8, 9); no finding.
- Card order: no card uses a symbol a later card creates; the PATTERN amendment lands in card 5 after the carry-over cards, and I found no defect in the shared decision.
- Test economy: cards 1 to 3 and 7 to 9 extend existing tables or files except the new `carryover_test.go` and `wiring_carryover_integration_test.go`, which cover behavior no existing test covers; no finding.
- Verify coverage: `## verify:` runs `./...` untagged and under `integration` in the template's order; no finding.
- `Custom` is not used, and each `Edit` card's `ImpactSummary` states a blast-radius conclusion; no finding.
