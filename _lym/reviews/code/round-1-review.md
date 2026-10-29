---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/gatecli/scopes_integration_test.go:431
    summary: No test row fails if notRunTiers counts a vet run as running its tier, the behavior D1 names.
    fix: In the passing --card subtest that plans vet runs under tier1 (the row around line 431), assert that stdout holds the not-run line naming tier1 among its tiers, using notRunSuffix as the other new assertions do; this fails if the vetStep exclusion in notRunTiers at internal/gatecli/tiers.go is dropped.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/gatecli/run.go:208
    summary: printNotRunTiers re-implements the tier-name join that tierNames in scope.go already does.
    fix: In printNotRunTiers, drop the names slice and loop and build the line from tierNames(numbers) joined with the gate test prefix and notRunSuffix.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/gatecli/run.go:208
    summary: The printed line omits the "passed <run tiers> only;" clause that D1 of the decision record specifies.
    fix: Either print the clause, by passing the tiers the plan ran (non-guard, non-vet runs) to printNotRunTiers and updating the line and its two test expectations in scopes_integration_test.go, or leave the line as is and state the shorter wording in internal/gatecli/doc.go; the doc already describes the shorter form, so the smaller change is to accept it.
---

### [LOW] Vet exclusion in notRunTiers is unpinned

**Location:** internal/gatecli/scopes_integration_test.go:431
**Issue:** D1 requires that a vet run under a tier's tag does not count as running the tier. No row asserts it: the added assertions cover the pass with tier1 named and a covered tier left out, but the passing `--card` row with vet runs does not check the line. Dropping `!run.vetStep` leaves the suite green. The `!run.guardStep` exclusion is likewise unpinned but low risk, since the guard group in these rows belongs to a listed tier.

### [LOW] Duplicate tier-name join

**Location:** internal/gatecli/run.go:208
**Issue:** `printNotRunTiers` builds the comma-joined tier names with a loop that `tierNames` at `internal/gatecli/scope.go:345` already performs with the same body.

### [LOW] Line wording differs from the record's D1

**Location:** internal/gatecli/run.go:208
**Issue:** D1 gives `gate test: passed <run tiers> only; <not-run tiers> did not run, so this pass is not green for them`; the code prints `gate test: <not-run tiers> did not run, so this pass is not green for them`. The board entry only asks the result to state the higher tiers were not run, and the record's wording is the writer's own claim, so the deviation is LOW. The acceptance criteria hold, and `doc.go` and the spec describe the shorter form.

## Rejected

- Lens code-tests, finding 1 (no not-run-line assertion under `--all`, explicit targets and `--gate diff`): the D2 acceptance criterion asks for a passing call under another scope form to print no line, and the computed-form assertion at `scopes_integration_test.go:185` pins that; one more row per form would repeat the same guard.
- Lens code-tests, finding 3 (manual tier has no row): `notRunTiers` has no manual-specific branch, so a manual row would exercise the same iteration the existing rows already pin.
- Lens code-correctness, cancelled-call race (line printed beside a non-zero exit after a signal): the line is accurate for the runs that passed, and the exit-code change belongs to `exitAfterSignal`, not the diff.

## Judged correct, no finding

- Focus 1: the line prints only under `code == 0 && sc.form == scopeCard`; a failing call and a halted call (return at `run.go:169`) print nothing.
- Focus 2: the not-run set is the declared tiers minus listed tiers and tiers a listed run covers; manual tiers are counted, and vet and guard runs are not.
- Focus 3: the line changes no exit code, record or green-record entry and goes to `request.stderr`.
- Focus 5: `doc.go` and the Tier 1 bullet state the line, its tiers and its bound, and agree with the code.
- No PATTERN change is declared and none is owed; `docs/overview.md` needs no edit since no module row or execution stack changes.
