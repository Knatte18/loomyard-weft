```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the two proposals that were carried out as proposed: F11 (card 9's shed test) and F12 (card 9's dobby walk).

**F11.** No case found. The planned test holds a run lock, and the walk only takes the hub lock (`internal/hubreconcile/walk.go:56`), so the two don't contend. The test arms through the full wire, and its moved-key-versus-busy assertion can fail when it should.

**[MEDIUM] F12: the new walk in `hubDobbyDeps` reaches existing tier1 tests that card 9 doesn't name, and no test covers the dobby walk.**
- Changed text: card 9, the bullet "`hubDobbyDeps` calls it with `prime` before it builds the dobby's deps", plus `internal/beamcli#hubDobbyDeps` in **Edit:**.
- The proposal puts the walk in `hubDobbyDeps`. That function is the production entry (`internal/beamcli/cli.go:715`, `runDobby`), and it is also called directly by many tier1 tests:
  - `internal/beamcli/dobbydeps_integration_test.go:60, 93, 236, 263, 289, 297, 413, 479, 565`
  - `internal/beamcli/dobbynoticestamp_integration_test.go:71`
- `hubforge` deletes the clone's build stamp on purpose (`internal/hubforge/hub.go:360-365`: "Removing the stamp makes a start verb driven in a test walk the hub"). So each of those calls now runs a full walk: hub lock, board pull, hub-wide commit, possibly an orch notice. That changes what those tests set up and may change what they assert, such as notice counts and Bolt state.
- Card 9 adds none of those tests to **Edit:**, gives them no row, and adds no row asserting that a dobby start walks. D5's dobby route therefore ships with no test.
- One concrete fix: move the call into `hubDobbyDepsStarting`, or into `runDobby`/`runDobbyWith`. Then add one `dobbydeps_integration_test` row: on a stale stamp, with a pair's `tapestry.yaml` holding an old key, the deps build and the file is moved. Name any test that now walks.

Tool calls made beyond the inherited context:
1. grep for `hubDobbyDeps` callers and `hubreconcile` locks: needed to know who calls the function the fix edits.
2. grep for `hubforge` stamp handling and read the dobby entry in `beamcli/cli.go`: needed to know whether test hubs carry a stale stamp, and how the dobby process reaches `hubDobbyDeps`.
3. grep for the locks `hubreconcile` takes: needed to rule out the walk contending with the run lock that F11's test holds.
