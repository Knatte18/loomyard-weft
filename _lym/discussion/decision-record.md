## Goal

Re-measure tier 1 and internal/fabricengine after the fabric test speed tasks land

Takes: every other part landed on main, and the report's W1 figures of 75.0 s and 57.0 s
Delivers: one lym gate test --tier 1 --all -- -count=1 pass and one lym gate test --tier 1 ./internal/fabricengine -- -count=1 pass on a quiet neve, read against W1 and the expected 36 s, 56 s and 48 s, filed as a board note naming the two commands, the expected times and whether the hub's tiers carry {test_targets}
Exports: nothing

## Scope

The part changes these packages:

- docs/research/speed (about 60 changed lines)

## Decisions

### D9 — Task order

Each line is one darn task, sized for one darn run, with what it waits for.

1. D1, the `dotgit` worktree reader: waits for nothing.
2. D2, registered templates: waits for nothing.
3. D3, the slimmer copy: waits for task 2, which edits the same file.
4. D4, quiet config: waits for nothing.
5. D5 without the `internal/hubreconcile` sites: waits for nothing.
6. D6, the folds: waits for nothing.
7. D7 `internal/fabricengine` anchor and origin: waits for tasks 2 and 6.
8. D7 `internal/fabricengine` live state and merge pairs: waits for tasks 2 and 6.
9. D7 `internal/hubreconcile`, with D5's sites there and D8's move: waits for task 2.
10. D7 `internal/loomcli`: waits for task 2.
11. D7 `internal/battencli`: waits for task 2.

Rationale: the folds edit the tests the `internal/fabricengine` moves rebuild, so they land first and each fold is judged against today's fixtures.
The rest of the pairs edit disjoint files: D5 alone edits `internal/fabricengine/replay_integration_test.go`, and D2's constraints on a build hold whether D3 lands before or after a move.

### D10 — The re-measure

Once every task has landed, one whole tier 1 pass, `lym gate test --tier 1 --all -- -count=1`, and one `internal/fabricengine` pass, `lym gate test --tier 1 ./internal/fabricengine -- -count=1`, on a quiet neve, read against the report's W1: 75.0 s and 57.0 s.
The expected package time is about 36 s.
The expected pass is about 56 s from these tasks alone, and about 48 s once the hub's tier commands carry `{test_targets}`.
`gate-tier-binaries-only` landed on `main` as `3aca2e50`, with `{test_targets}` opt-in and the hub's tiers not yet switched.
The split files the re-measure as a board note.

Rationale: two runs answer whether the package fell below the 35 s at which the other packages set the wall.

### Added after Discussion (parent, 2026-11-07): The writer runs both timed passes itself

Decision: Run the two passes the record names yourself, each once, in the foreground with the Bash tool timeout 3600000: 'lym gate test --tier 1 --all -- -count=1' and 'lym gate test --tier 1 ./internal/fabricengine -- -count=1'. This decision overrides the darn prompt's ban on lym gate test for this run: a docs run exists to run measurements, and the orch never runs one. Before each pass record the load average and the runs in flight from 'lym beam status', and state in the report that neve was not quiet, so the numbers are read against W1 with that noise named. Also report whether this hub's gate.yaml tiers carry {test_targets}. Commit the report; file no board note in place of the measurement.

Rationale: Trials run as darn docs runs in this hub (operator); waiting for a quiet machine has no end while the pipeline runs, and a timing with its load stated is still worth comparing.

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

- `gate-tier-binaries-only` landed on `main` as `3aca2e50`, with `{test_targets}` opt-in and the hub's tiers not yet switched; the D10 pass depends on whether the hub's tier commands carry `{test_targets}`, and D10 gives both expectations.

## Acceptance criteria

- D9: Review: the split's darn tasks match the order and dependencies of D9.
- D10: Review: the split files the re-measure note, naming the two commands and the expected times.
