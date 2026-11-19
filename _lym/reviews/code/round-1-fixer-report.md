---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed-differently
    reason: The proposed integration row needs a slot, build or run failure that no test can cause deterministically after the changed functions are known. The step's recording moved into mutationStep.recordFindings, and TestMutationStep_RecordFindings pins a failed and a cancelled call with no Functions or FileHashes. Partial counts are kept in the report.
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
  - id: F9
    disposition: fixed
  - id: F10
    disposition: fixed
  - id: F11
    disposition: fixed
---

# Fixer report, round 1

## Changes

Commits are grouped by file, since several findings share files.

- **F1** (`internal/gateslot/config.go`, `internal/gatecli/mutation.go`): `Config.HasGoCoverageMaps` reads a `scoperHasGoCoverageMaps` table, and `mutationStep.run` asks it instead of comparing with `go`.
  The existing literals in `reach.go`, `redundancy.go`, `mutate.go` and `reachverb.go` moved onto the same lookup.
  `TestConfig_HasGoCoverageMaps` pins the table.
- **F2** (fixed differently): see the disposition's reason.
  `mutationStep.answer` now calls `recordFindings`, which records the error and the partial counts, lists and not-judged packages, and returns before `Functions` and `FileHashes`.
  The review proposed an integration row in `TestGateMutate`; I made a tier 0 table test in `internal/gatecli/mutation_test.go` instead.
- **F3** (`internal/gateslot/doc.go`): the non-tier holder list, the bound paragraphs and the memory sentence name the mutation report step, and `mutation_max_mutants` is described as spread evenly when the cap is short of one each.
- **F4** (`internal/gatecli/mutate_integration_test.go`): rows for a red tier, a call outside a hub, a non-go scoper and a worktree with no parent branch, each asserting the note and the single summary line.
  A full `runTest` call over a `tier0` command of `false` asserts exit code 1 and a red-gate report.
- **F5** (same file): a gate call row over a diff that changes `Scale` and `Spare` asserts one survivor in `Scale`, the uncovered block of `Spare` and the counts.
- **F6**: the report-step row commits a change to `Clamp` and asserts `MutationReportMatches` is false at the new HEAD.
  `TestMutationReportMatches_RefusesAReportThatJudgedNothing` covers a note, a red-gate and a prime report.
- **F7** (`internal/testcover/mutant.go`): `logsOnly` refuses an `if` whose condition holds a call, through `holdsCall`.
  The comments in `mutant.go`, `linemutant.go` and `doc.go` say so.
- **F8** (`internal/testcover/linemutant_test.go`): the fixture gains an if-else, an else-if chain, an init-statement if, a literal-first comparison, a parameter named `runtime` and an if whose condition calls a function.
  Five rows and the every-line row cover them.
- **F9** (`internal/loomcli/mutationnote_test.go`): rows for a non-red `Note` and for a HEAD match error.
- **F10** (`internal/gatecli/mutatecheck.go`, `_lym/landing/summary.md`): both texts say the cap's mutants are sampled evenly across the changed functions.
  The summary file is not under git by this fixer.
- **F11** (`internal/gatecli/mutationreport.go`, `internal/gatecli/doc.go`): the non-Go file bound is stated on `MutationReport`, on `MutationReportMatches` and in the Mutation report section.

Gate repair: the gate flagged two wrapped comment breaks, in `internal/gateslot/doc.go` (the memory sentence) and `internal/testcover/doc.go` (the log-only sentence).
Both are rewritten as whole sentences, one per line.

Reach gate: the second gate listed the tests that reach each changed function and flagged `fillMutateAnswer` and `freshMaps` as reached by no mapped test.
Both are the `lym gate mutate` verb's existing code, which the round did not change in behaviour, and `TestGateMutate` runs them through the real verb.
No test was added, since a new test there would only repeat that row.

Verification: `lym gate test` over `internal/testcover`, `internal/gateslot`, `internal/loomcli` and `internal/gatecli` passed at tier 0, and `TestGateMutate` with the reach and redundancy integration tests passed at tier 1.

## Self-check

- fix-consistency, LOW, comment lines of 120 characters or more: fixed.
- fix-consistency, LOW, twin `Scoper != "go"` literals left: fixed, the four sites moved onto `HasGoCoverageMaps`.
- fix-consistency, LOW, `_lym/landing/summary.md` stale beside the new tests and table: fixed.
- fix-consistency, LOW, two long lines in the summary: fixed.
- fix-code-correctness, LOW, `holdsCall` also matches builtins and conversions: no change.
  The review's own fix says an `if` is log-only only when its condition holds no call expression, so the conservative reading is the proposal.
  Narrowing to builtins needs type information that the parse-only planner lacks, and a shadowed `len` would defeat a name check.
- fix-proposal, LOW, the same `holdsCall` over-rejection: no change, same reason.
- fix-proposal, LOW, the table leaves `config.go:329` listing scoper names separately: no change.
  `TestConfig_HasGoCoverageMaps` pins the three accepted names, and a name missing from the table reads as false, the old `!= "go"` answer.
- fix-proposal, LOW, the red tier row calls the step directly: fixed with a full call row asserting exit code 1.
- fix-affected-set, LOW, `linemutant.go` comment stale: fixed.
- fix-affected-set, LOW, existing literals left silent: fixed, see the twin literals above.
- No fork returned without a report.

## Disputed

None.

## Deferred

None.

## Rulings

None.
