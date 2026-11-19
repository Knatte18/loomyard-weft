# stencilstore: refuse a board stencil copy a newer lym binary wrote

A long-running driver can outlive the binary that wrote the board's stencil copies.
After `update-plugins` refreshes the board stencils, the old process renders text it does not understand.
It then fails mid-run with a bare template error, such as `include "loom-fragment-loosened-check" has no entry in the includes`.

The missing-include wiring from the issue is already fixed on main, with its tests in `loomcli` (`TestFixerSpec`) and `gatecli` (`TestRunWithFixer`).
This change fixes the cause that surfaced it: the version skew.

Closes #149

## What changes

- `stencilstore.Read`, and the include and engine-block reads behind `ReadForEngine` and `ReadSet`, return a `*NewerBoardCopyError` when the board copy was written by a newer binary.
  The error names the stencil, the writer's revision and the running revision.
  It gives the way forward: restart the run's driver on the current lym binary, and update that binary first if it is older than the board.
- A refusal never writes the board.
- A copy whose body equals the running binary's shipped default is read as it stands.
  A stencil the binary does not ship, and a copy with no recorded writer, are never refused.
- `contracts/stencils` registers its embedded defaults at init through `stencilstore.RegisterShipped`.
  The defaults are only compared against; no read renders from them.
- `cmd/lym/stencilseed.go` registers the build-ancestry ordering the refresh already uses, through `stencilstore.RegisterOrdering`.
  A writer counts as newer when that ordering answers `RecordedNotOlder` and its revision differs from the running one.
  Without a registered ordering, commit time decides.

## Design decisions

- Fail early rather than fall back to the embedded stencil.
  The board owns stencil copies, and an operator's edits there must not be silently bypassed.
- An edited copy from a newer writer is refused the same as an untouched one.
- The body comparison runs before the ordering, because the ordering may run git.
- `RegisterShipped` panics on a second call, since the registry is process-global and set once from package init.
- The process-global state is exposed to tests through `SetSkewStateForTest` in `export_test.go`, so the tests run through the public read paths.
  Tests that call it cannot be parallel.

## Tests

`TestReadRefusesBoardCopyFromNewerBinary` in `internal/stencilstore/skew_test.go` covers:

- top-level, plain-include and engine-block reads of a newer writer's copy;
- an edited copy from a newer writer;
- older, equal-time, absent and shipped-body writers, a running binary with no commit time, and an unshipped stencil;
- descendant and ancestor writers under a registered ordering, and an unknown ordering.

Every case also checks that the board file is unchanged.
The tests have not been re-run since this description was written.

## To check by hand

- A copy a newer writer edited, read through a path that reaches the board copy indirectly (`readInclude` in `engineblock.go`): confirm the refusal covers every such path.
- The embedded-defaults registration in `contracts/stencils/stencils.go`: confirm that comparing against shipped defaults at read time does not conflict with the board owning stencil copies.
- The skew tests in `skew_test.go` reach the logic through `SetSkewStateForTest`: confirm they exercise the public reads rather than the unexported helper alone.
- The writer ordering in `skew.go` against the refresh ordering in `cmd/lym/stencilseed.go`: confirm the two agree for descendant builds.
