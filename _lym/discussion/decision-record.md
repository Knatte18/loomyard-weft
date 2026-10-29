# Decision record — webster-recover-batch-flake

## Goal

`TestRecoverSpawnOrAttach` in `internal/websterengine/recoverbatch_test.go` passes or fails on the code alone, never on how its parallel rows interleave.

## Scope

In scope: the record the `uncheckableRefusal` setup-and-check pair shares through a closure variable, which every row reusing one pair reassigns from its own parallel subtest.
The `atStartSetup` pair is reused by four rows and the `committedSetup` pair by two, so a row's check can compare its fixture's record against another row's, which reports "record changed" over an unchanged record and is a data race on the variable.

Out of scope: `internal/websterengine` production code, and the other closure variables in the test that a single row reads.

## Decisions

### D1 — The seeded record lives on the subtest's fixture

The refusal setup stores the record it seeds on the `recoverFixture` it is given, and the refusal check compares against the record on its own fixture, with the closure-shared variable removed.
Rationale: the fixture is built per subtest, so each row compares against the record its own setup seeded, and the check keeps its pointer-identity test that the refused spawn did not replace the record.

## Constraints

- `PATTERN-test-speed`: rows keep `t.Parallel`, and the test stays at its current tier with no new test.
- `PATTERN-test-isolation`: the fix stays inside the test files of `internal/websterengine`.

## Auto-mode assumptions

- A darn run holds no interview; the board entry's diagnosis is taken as confirmed by reading the rows that call `atStartSetup` and `committedSetup`.

## Open risks

- The race detector's availability through `lym gate test` decides whether repeated race runs can be shown; the gate's own runs are the evidence used.

## Acceptance criteria

- D1: every row of `TestRecoverSpawnOrAttach` that reuses one refusal pair compares against the record its own setup seeded, and no closure variable is written from more than one row's subtest.
- D1: the existing refusal rows keep their assertions, including that the refused spawn leaves the same record in place, and the test passes under `lym gate test`.
