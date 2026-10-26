# Fixer-report — code review round 1

Every finding was fixed; each fix is its own commit.

## Changes

- **F1** (fixed as proposed): `contracts/stencils/stencils.go` gains `"plan-fixer": {"webster-template-plan-fix"}` in `roleOpeningStencils`, beside `verify-fixer`.
- **F2** (fixed as proposed): the `runwrites_test.go` subtest that seeds `VerifyFixers` also seeds `PlanFixers` with session `p1`; its audited-sessions assertion fails if `p1` is ever audited.
- **F3** (fixed as proposed):
  - `batchpause_test.go` gains a `fix-plan` covering-pause case and its dispatch.
  - The refusal table in `planfixer_test.go` gains rows for HEAD, status, strand start and run lookup failures, each asserting the transient way forward text and that no record is appended.
  - The table's `arrange` now receives the fixture, and a `wantAdded` field records the AddStrand calls the fake saw: the start-failure row records its attempted call and the run-lookup row its added-then-stopped strand, so both want 1.
- **F4** (fixed as proposed): a `webster-template-plan-fix` row in the marker table of `contracts/stencils/markers_test.go`, listing the optional `friction_directive` and `parent_directive` markers as the sibling cold-fixer row does, and a `plan-fix` case in the friction directive table of `template_test.go`.
- **F5** (fixed as proposed): the four static stencil strings are removed from the spawn subtest's `want` list.
- **F6** (fixed differently): `ReadConflictRecord` takes its findings from a new `findingLines(text)` that `readFindingLines` also uses, instead of calling `readFindingLines`.
  The proposed call would read the record file twice, and `ReadConflictRecord` already holds the text it needs for the batch header.
  The `Cause` field and its parse are dropped, as proposed.
- **F7** (fixed as proposed): `fix-plan` is added to the verb lists in `wiring.go`, `batchpause.go` and `batchpause_test.go`, and to `cli.go:17`, a twin the review did not name.

## Self-check

- fix-consistency, LOW: `runlevel.go:777` verb list omitted `fix-plan`; fixed, with the adjacent comment at `runlevel.go:775`.
- fix-code-correctness: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Verification

`lyx gate test` passed for `./contracts/stencils` and `./internal/websterengine` at tier 0, and for `./internal/webstercli` at tier 1.

## Disputed

None.

## Deferred

None.
