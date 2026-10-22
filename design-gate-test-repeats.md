# Test runs repeat at the end of a loom run: tier0 inside every tier, the suite four times, tmux only at Publish

`gate-test-repeats` · Note · enhancement, gate, loom, webster, fabric

## Seen (2026-10-22)

The loomyard hub's tiers were first declared on Bolt on 2026-10-22 (`d5a60908b`): `tier0` untagged, `tier1` integration (with `go build ./...` and `go vet` under the integration, tmux and llm tags first), `tier2` tmux, `tier3` llm by hand only.
The gate lists are `card: [0]`, `batch`, `round`, `final` and `landing: [0, 1]`, and `publish: [2]`.

Three things make a loom run's test runs clumsy:

- **`tier0` runs again inside every higher tier.**
  A Go build tag only adds files, so `go test -tags integration` and `go test -tags tmux` both compile and run every untagged test; `covers: [0]` only stops a call from running `tier0` a separate time.
- **The whole suite runs up to four times at the end of a loom run.**
  Webster's `final` gate, Merge-In's `landing` gate, Publish's `landing` gate plus `publish`, and Finalize's `landing` gate each run `tier0` and `tier1` over the whole tree.
  Since `905bc88df` a re-run over an unchanged tree should take its results from Go's test cache, so only a tree that changed, such as after Merge-In takes a moved parent, runs for real; how close to free that is in practice is not measured, and each run still waits for a slot, starts up and writes a log.
- **The tmux tier runs only at Publish.**
  `PATTERN-test-speed` gives it its own tier because it needs a real tmux server and was slower, but the measurement below has it faster than `tier1`.

### Measurement (fork, 2026-10-22, Neve, `de6b8465f`)

- `tier1 --all`, cold (first run of the commit on the machine): 114 s wall; about 70 to 75 s of it was build, the three vets and test compiles.
- `tier1 --all`, fully cached: 28.8 s, about 20 s of it the fixed build and vet cost.
- A normal `tier1` run after a change: about 60 s, about 20 s build and vet plus about 40 s tests.
- `tier1` package time 125.9 s at `-p=4`: about 20 s is the tests `tier0` also runs, about 106 s the integration-only tests.
- `tier0 --all`: 22.5 s wall.
- `tier2 --all`: 60 s wall with a warm build cache, about 45 s of it tests.
- Slowest `tier1` packages: `internal/fabricengine` 37.2 s, then `battencli` 8.8 s, `cmd/lyx` 6.9 s, `gatecli` 6.4 s, `webstercli` 5.8 s.
  `fabricengine` alone sets the length of `tier1`'s test phase.
- Slowest `tier2` packages: `loomcli` 44.7 s, `reedengine` 37.9 s, `reedcli` 37.4 s.

## Idea

- **Disjoint tiers.**
  Mark `tier0` test files so a higher tier does not run them again.
  Orch recommendation, not a decision: a negated tag such as `//go:build !integration && !tmux`, which plain `go test` and gopls still see, over a positive `//go:build tier0`, which hides the tests from both; either touches every untagged test file, and a new tier tag must join the negation.
  The measured gain is small, about 20 s of package time per higher-tier run.
- **A green run carries forward.**
  A gate runs its tiers only when the tree changed since the last green run of the same tiers, instead of relying on Go's test cache to make the repeat cheap.
- **Run tmux more often.**
  The tmux tests run in a batch or final gate, or share a tier with integration, instead of only at Publish.
  The tmux tests have been the flaky ones in gates (task `darn-verify-fixes`), which a move must weigh.
- **Speed up `internal/fabricengine`'s tests**, the largest single item in `tier1`.

## Operator view, not a decision

- The operator pictured every tier tagged by its own name, `tier0` included, so no test runs in two tiers (2026-10-22).
- tmux is an important part of this project, so running its tests less often is not good (2026-10-22).
- Suites running several times at the end of a loom run is clumsy (2026-10-22).
