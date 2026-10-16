# Reed down ends its watchdog; agents resolve lyx to the spawning binary

`lyx reed down` of a hub's last session now leaves no reed watchdog behind, and the Bash tool of every Claude agent the Claude shuttle engine spawns resolves bare `lyx` to the spawning binary.

Closes #679
Closes #680

## What changes

### Watchdog ends with the hub (#679)

- `reedengine.Engine.Down` writes a stop marker into the told discover signal file when it finds no session left and ends the server, whether or not the server then confirms gone.
  Standalone mode and a `down` that leaves sibling sessions write nothing.
  `Down`'s result now reports whether it ended the server.
- The watchdog daemon in `internal/reedcli` reads the signal file's content before removing it.
  A stop-woken cycle skips the 5-second cadence floor, but never runs sooner than 1 second after the previous cycle.
  If its listing is idle the daemon exits at once; if affirmative it carries on as an ordinary discover.
  A cold-boot touch truncates the marker, so a racing `up` turns a pending stop back into a discover.
- On every exit (stop, idle-cycle rule, idle retire) the daemon releases its lock and lists the hub's sessions once more.
  An affirmative listing spawns one successor through `reedengine.SpawnWatchdog`; a cancelled context spawns nothing.
  This closes the window where a session boots while the daemon is exiting.
- The binary-replaced check moves after the cycle's session listing.
  A replaced binary with an idle listing ends the daemon without a successor of its own, so a redeploy no longer revives a watchdog for an empty hub.
- `lyx reed down` in hub mode polls the watchdog's single-instance lock for at most 10 seconds after `Down` reports it ended the server.
  On timeout it logs one Warn naming the hub and the lock path and still succeeds, with an unchanged envelope.
  The bound and poll interval are injectable.

### Agents resolve lyx to the spawning binary (#680)

- The Claude engine's `bash-env.sh` keeps `exec </dev/null` and now prepends `LYX_BIN`'s directory to `PATH` when `LYX_BIN` is set.
  The value is quoted, and it is read per Bash command, so a resumed session follows the binary its pane's latest prelude exported.
  On Windows the directory goes through `cygpath -u`, and the statement does nothing when `cygpath` is absent.
- The `LYX_BIN` key moves from an unexported constant in `internal/reedengine` to an exported one in `internal/agentname`, beside `StrandNameEnv` and `ParentEnv`.
  Reed's prelude and the Claude engine read the same declaration.
- `internal/reedengine/doc.go` documents the pane shell's resolution as built.
  The sandbox suites' `command -v lyx` pre-condition bullets name the shell the check runs in.

## Design decisions to know

- A fresh shell that an operator-supplied strand command starts (`--cmd bash`, a plain `--cmd claude`'s Bash tool) re-runs the operator's rc after reed's prelude, so its bare `lyx` follows the rc's PATH.
  Reed has no engine-neutral hook that runs after a child shell's rc, so this departs from the board entry's "even when the shell rc prepends another directory" for those shells.
  The creel suite's plain Claude and bash strands are such shells, so its `command -v lyx` bullet now says the check does not hold there and points at `SANDBOX-SHUTTLE-SUITE.md` for the agent Bash tool check.
- `down` of a non-last session leaves the watchdog running, since it serves the siblings.
  `reedengine.EndSessionByName` writes no stop; a daemon it leaves on an empty hub exits by the existing idle-cycle rule.
- `sessionsAreIdle` counts a list-sessions error as idle, so a transient error on a stop-woken cycle can end a daemon whose hub still has a session; the exit re-check bounds that to one successor spawn.
- A free lock means the daemon has left its loop; the exiting process may still run its one re-check listing and at most one successor spawn when `down` returns.
- A daemon whose file watch failed to open never sees the stop and exits by the idle-cycle rule as before.
- `cmd/lyx/testdata/test-budget.yaml` is raised for the new tests.

## To check by hand

- The `down` lock-wait poll in `internal/reedcli/watchdog.go` and `up.go`: a review flagged it as lacking the annotation the repo's scans expect (finding `watchdog-down-poll-unannotated`).
  Confirm `lyx gate test ./internal/reedcli` and the pattern scans accept it.
- Shuttle suite, on a machine whose rc prepends another `lyx` (the reporter's `~/.bashrc` prepends the Go bin dir): `command -v lyx` from a spawned agent's Bash tool must name `LYX_BIN`.
  This is the live detector if a Claude Code release runs `CLAUDE_ENV_FILE` before its shell snapshot, which would stop the prepend from winning.
- Reed suite, after `lyx reed down` of a hub's last session: no reed watchdog process remains, including after redeploying the dev binary.
- A `down` that lands during an in-flight orphan reap can hit the 10-second bound and warn, since the daemon finishes the reap before exiting.
- `creel-stage1-fixes` edits other lines of `SANDBOX-CREEL-SUITE.md`; this change touches only the pre-condition bullet and the closing sentence after the bullets, so whichever lands second merges the other's lines.
