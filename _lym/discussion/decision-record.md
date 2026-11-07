## Goal

Fold the overlapping internal/fabricengine tests the tier 1 report read

Takes: today's fixtures and the report's section 4 rows with their alone-pins column
Delivers: TestVerbCases_CleanState's preconditions in runCell, and the TestAnchorChange, TestStoreHousekeeping, TestReconcileOrigin_Moves, TestPushFailureClasses and TestBolt_PullThenCommitWritten folds, each commit naming the assertions its folded row alone made
Exports: nothing

## Scope

The part changes these packages:

- internal/fabricengine (about 500 changed lines)

## Decisions

### D6 — Folds of overlapping `internal/fabricengine` tests

- `TestVerbCases_CleanState` goes: its three preconditions, Add's launcher blocker is a directory, Checkout's branch differs and Pull's bare advanced, move into the cross product's `runCell` in `internal/fabricengine/livestate_matrix_test.go`, checked right after the verb's arrange and before the state is applied.
- The report's section 4 rows that it read or found again by name fold as it names each one:
  - The three setup-free refusal rows of `TestAnchorChange` share one `hubforge.CopyHub`.
  - The two probe subtests of `TestStoreHousekeeping` fold into the gc-prune row.
  - `owned_elsewhere` of `TestReconcileOrigin_Moves` runs its six replies on one pair.
  - The paired settle and backoff rows of `TestPushFailureClasses` assert on the first push, and the backoff rows become one table.
  - The three subtest pairs of `TestBolt_PullThenCommitWritten` become two-row tables.

Each fold keeps every assertion its row alone made, named in the fold's commit, and the report's "alone pins" column is the list a review checks.

Rationale: about 6 s of the package's serial time, about 2 s of wall, with no assertion lost.

## Constraints

- `PATTERN-leaf-packages`: the D1 reader keeps `internal/dotgit` standard-library only and spawning nothing; the background's list of what `dotgit` parses gains the worktree entries and refs.
- `PATTERN-gitrepo-client-boundary`: D1 is a local read, which the entry gives to `dotgit`; the porcelain fallback stays on `gitexec`.
- `PATTERN-fabric-boundary`: D1's change to `fabricengine.List` stays inside `internal/fabricengine`; `internal/websterengine` reads its own worktree's siblings through `dotgit`, as it reads them through git today, and through its present porcelain read on a reftable repository.
- `PATTERN-test-isolation`: every template is a hub `fabriccli.CloneAndWire` built plus the build function's steps; no hub is hand-assembled, and no test package wraps `hubforge` in its own fixture type.
  The background's hub-fixture paragraph gains the registered template.
- `PATTERN-test-speed`: each test keeps one `tiergate.Require` at the lowest tier that fits; D8 follows it.

## Auto-mode assumptions

- D1: a reftable repository falls back to the porcelain path.
- D3: the binding fold stays on git, for the reflogs, in place of the entry's "no `amend` and `push --force` per copy".
- D4: the per-repository config layer stays.
- D6: the survey's rows the report did not read again are out.
- D7: `newMergePairFixture` moves only for repeated layers, and the `internal/battencli` `beamHub` joins the moves.

## Open risks

## Acceptance criteria

- D6: Review: each fold's commit names the assertions its folded row alone made, and each stands in the remaining test.
