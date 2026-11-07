# loomcli: beamHub and the bad-reed fixture onto registered templates

Two `internal/loomcli` fixtures built their whole start state on every call.
Each now copies a `hubforge.Registration` template, built once per test binary.

- `beamHub` (`projectrecord_integration_test.go`), used by six tests and subtests, copies `beamHubRegistration`.
  Its build is the fixture's former steps, moved unchanged: the beam pair `the-beam`, two pairs forked from it, and the uncommitted `board.json` and shortname record on the board.
- `newBadReedUpFixture` (`smoke_helpers_test.go`), used by four sites, copies `badReedUpRegistration`.
  The base is the template: the providerless config seed with the corrupted `mouse:` reed line, then the pair `loom-smoke-task`.
  Each site's seed function still runs on its own copy, after `lymcwd.Resolve`.

Because the builds are the former steps moved as they were, each test starts from the same state and keeps its assertions unchanged.

## Design decisions

- Both builds start from the `.` shape with no pairs.
  `SeedConfig` must commit on the prime records before the pair is added, so the pair forks from the seeded config.
  For that reason the pair is added in the build and not listed in `Base.Pairs`, which `hubforge` adds before `Build` runs.
- `newWiredPairFixture` stays on `CopyHub`.
  It has two sites, below the three-site threshold the decision record sets.
- Neither build makes a process-wide call.
  Both write only inside the template root and push only what `AddPair` pushes on the board.
  `TestRunOwnerVerdict` registers the board's Bolt replayer in parallel with these builds.
  A replayer runs only after a rejected push, and nothing else pushes into a template's own bares.

## To check by hand

- Compare the tier 1 time of `internal/loomcli` against `main`.
  The gain is the build steps the six `beamHub` sites and four `newBadReedUpFixture` sites no longer repeat.
