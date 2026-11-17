---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
---

# Round 1 fixer-report

## Changes

All eight findings are fixed as proposed.
Commits: `testcover: fix F2-F8` (one commit for F2 to F8, which share `mutant.go` and `mutant_test.go`),
`gatecli: fix F1`, and a follow-up for the self-check findings.

- F1: `internal/gatecli/redundant_integration_test.go` fixture package `cov` gained `Allowed` (`||`), `Both` (`&&`),
  `Banner` over a `//go:embed banner.txt` string, and the tests `TestAllowedChecked` and `TestAllowedRun`.
  `TestAllowedChecked` is decided by one clause per chain and asserts both embedded lines it can reach.
  The test now expects 20 buildable mutants, the new run counts, and `TestAllowedChecked` refuted with its unique kills named by operator.
  Line numbers of the existing expectations shifted with the added code.
- F2: `planFixture` writes `fixvars.go`, a block-less file holding one package variable that two blocks (`Tagged`) read.
  A row targets only the later block and expects both blocks.
  The per-row `file` field names the file of the row's mutants.
  Block indexes are compared in index order, and `slices.IsSorted` pins the key order of `Mutant.Blocks` separately, since the keys sort by hash.
- F3: `markFixed` leaves a string-literal composite key unfixed; the `PlanMutants` doc says so; the fixture holds `map[string]int{"k": 1}`.
- F4: rows for a three-operand chain (`Chain`), a keyed array element in a package variable (`keyed`),
  and a local `names` in `Shadow` that shadows a package variable holding literals.
- F5: `PlanMutants` gives each package variable the reader blocks of every variable whose value names it, to a fixed point.
  `variableValueReads` finds those names. The fixture holds `base`, read only through `all`.
- F6: the `OperatorDeleteClause` comment, the `PlanMutants` doc and the package doc state the operand is still evaluated.
- F7: the `parsePackageFiles` doc states the default build context bound; GOFLAGS tags are not read.
- F8: `parser.ParseComments|parser.SkipObjectResolution`.

A later gate flagged six comment lines in `doc.go` and `mutant.go` that wrapped mid-sentence.
Each is now rewritten one sentence per line, with no change to its meaning.

Verified with `lym gate test --tier 1 ./internal/gatecli ./internal/testcover`: 313 passed, 0 failed.

## Self-check

- fix-code-correctness, MEDIUM: `lymMutantLine` index still unpinned, since only line 2 was asserted. Fixed: `TestAllowedChecked` asserts lines 1 and 2.
- fix-code-correctness, LOW: `lymMutantDropAnd` never switched on. Fixed: `Both` with an `&&` chain, one clause deciding.
- fix-code-correctness, LOW and fix-proposal, LOW: two comment lines at or over 120 characters. Fixed: split.
- fix-consistency, LOW and fix-affected-set, LOW: `Mutant.Blocks` and `doc.go` not stating the widened attribution. Fixed.
- fix-affected-set, LOW: `doc.go` lacks the evaluated-operand statement. Fixed.
- fix-consistency, LOW: only `OperatorDeleteClause` carries a comment in its const group. Disputed, every fix worse: a comment on each sibling restates its name.
- fix-proposal: no further findings.
- fix-affected-set: no findings beyond the two LOW items above.

## Disputed

- Self-check finding of fix-consistency (const group comment), ground `every fix worse`:
  the other operator names state what they do, and `OperatorDeleteClause` alone hides a behavior a reader would not expect (`internal/testcover/mutant.go:41`).
  A comment on each sibling would restate its name.

## Deferred

None.

## Rulings

None.
