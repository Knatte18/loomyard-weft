# Prime dobby starts lym gate testmap every test_map_every landings

The prime parent's dobby now keeps the per-test coverage maps fresh.
After every `test_map_every` landings it starts `lym gate testmap --max <test_map_batch>` on the packages stale at that moment.
What the cap leaves unbuilt waits for the next refresh.

## What changes

- `internal/dobbyengine`: a new tick step, `testMapStep`, in `testmap.go`.
  `countLandings` counts landings as the board tidy does, into a new `State.LandingsSinceTestMap`.
  Once the count reaches `test_map_every` the step starts the refresh through `Deps.StartTestMap` and zeroes the count.
- Two beam config keys, both in `template.yaml`:
  - `test_map_every` (default 5, next to `board_tidy_every`): 0 never refreshes, and a negative value is refused.
  - `test_map_batch` (default 5): the `--max` of each refresh; a value below 1 is refused.
- `internal/beamcli`: `hubDobbyDeps` fills the two seams for the prime's dobby only.
  `startTestMap` starts the refresh with the binary `selfexec.LymBinary` names, in the prime's anchor and its own process group.
  It registers the process under `runproc.HubOwner` as the new `runproc.RoleTestMap` (`test-map`), and the dobby waits on it in the background.
  `testMapRunning` reads that record.
- `internal/runproc`: the new `RoleTestMap` constant.
- `internal/orchcli`: `BeamDown` of the prime ends the refresh through `runproc.EndRole` after the dobby, whether or not the dobby was live.
- Docs: `docs/overview.md` and the package docs of `dobbyengine`, `beamcli`, `orchcli` and `fabriccli`.

## Design decisions

- The trigger is the landing count, not a prime-HEAD record or the refresh's exit record.
  No on/off key exists; `test_map_every: 0` is the off switch, and the step zeroes the count while it is 0.
  Enabling the key later therefore counts only later landings.
- No refresh starts while a live `test-map` record exists, so a restarted or re-exec'd dobby starts no second one.
  The count is kept and the refresh starts on the first tick after the live one ended.
- A failing start keeps the count and is retried on the next tick.
  A failing step is logged once per episode and does not end the tick, since the refresh is optional.
- A registration that fails is logged and leaves the refresh unregistered to end on its own; the start still counts as made.
- An anchor move of the prime waits for a running refresh, as it does for a pass in flight, so a move can wait out a whole build-and-test run.
  The dobby starts no refresh while the move stands.
- Bounds:
  - While a board-tidy agent is in flight no landing is counted, so a refresh can start up to `board_tidy_timeout_min` late.
  - A dobby that dies without a stop leaves the refresh running until the verb ends within its own timeouts.

## To check by hand

Nothing here was run against a real hub; the tests use stand-ins and launch no test binary.

- `lym beam stop` on the prime with the dobby already dead: `beamdown_test.go` pins that the refresh ends then.
  Check that a real refresh started by a live dobby is ended and leaves no `test-map` record.
- `startTestMap` registers the child after `cmd.Start`, so a crash between the two leaves an unregistered refresh.
  Check that this window is acceptable; it is logged and the refresh ends on its own.
- `starttestmap` tests in `internal/beamcli` cover the registration and the unregistered-start path;
  check that they hold on the platforms you run (the integration test is Unix-only).
