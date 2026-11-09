# The prime's dobby starts `lym gate testmap` every few landings

The coverage maps `lym gate testmap` builds go stale as landings change packages.
Until now, someone had to run the verb by hand.
With this change the prime's dobby refreshes them on its own, at a cadence counted in landings.

## What changes

- **The refresh step (`internal/dobbyengine/testmap.go`).**
  The board tidy's landing count now also adds each landing it counts to a new state field, `LandingsSinceTestMap`.
  When that count reaches `test_map_every`, a tick starts `lym gate testmap --max <test_map_batch>` and resets the count to zero.
  The verb builds the packages that are stale when it runs, at most `test_map_batch` of them;
  the next refresh picks up whatever the cap left unbuilt.
  The step runs after the board-tidy step, and like it, never while an anchor change stands.
- **One refresh at a time.**
  No refresh starts while a live `test-map` record exists.
  This holds for a restarted or re-exec'd dobby too, since it reads the same record.
  The count stays at or past the threshold, so the refresh starts on the first tick after the live one ended.
  A start that fails keeps the count and is tried again on the next tick.
- **Prime only.**
  Only the prime's wiring fills the two new seams, `Deps.TestMapRunning` and `Deps.StartTestMap`.
  A beam's dobby leaves them nil, so it counts nothing toward a refresh and starts none.
- **Spawn and registration (`internal/beamcli/dobbydeps.go`).**
  `startTestMap` starts the verb in the prime's anchor, using the binary `selfexec.LymBinary` names.
  The child gets its own process group and its output is discarded.
  It is registered under `runproc.HubOwner` with the new role `runproc.RoleTestMap` (`test-map`), and the dobby waits on it in the background and logs its exit.
  If the registration fails, the child is ended again through `runproc.EndProcess`, so no unregistered refresh is left running.
  `testMapRunning` reads `runproc.List` for a live `test-map` record under the hub owner.
- **Stop (`internal/orchcli/stop.go`).**
  `lym beam stop` of the prime ends the dobby, then the refresh, through `runproc.EndRole` with `runproc.RoleTestMap`.
  The refresh needs its own end: it has its own record, which `EndRole(RoleDobby)` spares along with the processes below it, and its own process group.
  The dobby is ended first so that it cannot start a new refresh after this end has run.
  A beam's stop is unchanged.
- **Config (`beam.yaml`).**
  Two new keys, both with defaults and loaded through `configengine`:
  `test_map_every` (default 5; 0 never refreshes; a negative value is refused)
  and `test_map_batch` (default 5; a value below 1 is refused, since the verb refuses `--max 0`).
  No existing key changes meaning, and there is no on/off key: `test_map_every: 0` turns the refresh off.

## Design decisions

- The trigger follows the operator's decision recorded in the Goal, which replaces D6's trigger:
  the dobby counts landings as the board tidy does.
  There is no prime-HEAD record and no trigger from the verb's exit record, so nothing reads that record's leftover count.
- The count reuses the board tidy's landing count instead of keeping a second set of seen slugs.
  The bound: while a board-tidy agent is in flight no landing is counted, so a refresh can start up to `board_tidy_timeout_min` late.
- The stop lives in `internal/orchcli`'s `endDobby` wiring, the path `lym beam stop` takes, rather than in the dobby's own shutdown.
  On a SIGTERM the dobby gets only `runproc`'s grace before SIGKILL, so an end run from the dobby itself might not complete.
  This puts a small change in `internal/orchcli`, a package outside the record's scope list.
- A dobby that dies without a stop leaves the refresh running.
  The refresh ends on its own within the verb's build and test timeouts.
- While the refresh runs it is a live child of the prime's dobby, so an anchor move of the prime reads it as a pass in flight and waits for it.

## Tests

- `TestTestMapStep` (`internal/dobbyengine/testmap_test.go`): a start every `test_map_every` landings with `test_map_batch` as the cap, and none at `test_map_every: 0`.
  It also covers the hold while a refresh is live, across a restarted daemon, followed by a start once it ended;
  a failed start that keeps the count and is retried;
  and no counting when the seams are nil.
- `TestLoadConfig` covers the new defaults, a `test_map_every` of 0, the refusal of a negative `test_map_every`, and the refusal of a `test_map_batch` of 0.
  The expected configs are now built from one defaults value, so a new key no longer means editing every row.
- `TestTestMapRunning` (`internal/beamcli`) reads live records against a temp run-process directory.
- `TestEndDobbyRoles` (`internal/orchcli`) checks that the prime's stop ends the dobby and then the refresh, that a beam's stop ends only the dobby, and that a failed dobby end does not go on to end the refresh.
