# Give each TestRecoverSpawnOrAttach row its own seeded record

Fixes the flaky `TestRecoverSpawnOrAttach` subtest in `internal/websterengine` that failed once with "record changed" over a record it printed unchanged (issue #902).

## Cause

`uncheckableRefusal` returns a setup and a check that shared the seeded batch record through a closure variable.
The `atStartSetup` pair is reused by four rows and the `committedSetup` pair by two.
These rows run as parallel subtests, so each one's setup reassigned the same variable.
The clean-tree row's check could then compare its fixture's record against a record another row had seeded, and the pointer comparison failed.
The writes were also a data race.

## Change

- `recoverFixture` gains a `Seeded` field holding the batch record a row's setup seeded.
  The fixture is built per subtest.
- The refusal setup stores its record on the fixture, and the refusal check reads it from there.
  The check's assertions are unchanged, including the pointer-identity check that a refused spawn left the seeded record in place.

Only test files change.

## Design decisions

- The record moved onto the fixture instead of each reused pair being declared once per row.
  The rows that compose `atStartSetup` stay as they are.
- `scratchPath` and `longBesidePathBase` stay as closure variables, since a single row writes and reads each.

## Testing

`lym gate test` over the diff scope is green, `TestRecoverSpawnOrAttach` included.
The board entry asked for repeated runs under the race detector.
`lym gate test` has no race or repeat option and a raw `go test` is refused in this environment, so those runs were not made.
