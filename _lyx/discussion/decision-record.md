# Decision record — reed down ends its watchdog; strand panes resolve lyx to the spawning binary

## Goal

Fix two reed defects from the creel stage 1 re-test of 2026-10-16:

- #679: after `lyx reed down` ends a hub's last session, the reed watchdog has left its loop and released its lock once `down` returns, and its process ends after one re-check listing; a redeployed binary never brings one back for an empty hub.
- #680: bare `lyx` resolves to the spawning binary in a strand's pane shell and in the Bash tool of every Claude agent the Claude shuttle engine spawns, even when the operator's shell rc prepends another directory holding a `lyx`.

## Scope

In:

- The hub watchdog daemon in `internal/reedcli` (its loop, its exit and its binary-replaced retire) and the `down` verb.
- `reedengine.Engine.Down`'s request that the watchdog stop, when `down` ended the hub's tmux server.
- The Claude engine's `CLAUDE_ENV_FILE` content in `internal/shuttleengine/claudeengine`.
- Moving the `LYX_BIN` env key's declaration to a package both reed and the Claude engine import.
- Docs: `internal/reedcli` and `internal/reedengine` `doc.go`, the `down` and `watchdog` verb help, the Claude engine's `doc.go`, and the `command -v lyx` pre-condition bullet of `tools/sandbox/SANDBOX-REED-SUITE.md`, `SANDBOX-SHUTTLE-SUITE.md` and `SANDBOX-CREEL-SUITE.md`.

Out:

- Ending the watchdog while sibling worktree sessions are still live on the hub: it serves them, so `down` of a non-last session leaves it running.
- `reedengine.EndSessionByName`, the pair teardown's route once the task worktree is gone: it writes no stop, and a daemon it leaves on an empty hub exits by the existing idle-cycle rule.
- A fresh shell that an operator-supplied strand command starts (`--cmd bash`, a plain `--cmd claude`'s Bash tool) re-runs the operator's rc after reed's prelude, so its bare `lyx` follows the rc's PATH.
  This departs from the board entry's "even when the shell rc prepends another directory" for those shells: reed has no engine-neutral hook that runs after a child shell's rc (D7).
  The creel suite's plain Claude and bash strands are such shells.
- The creel reminder's `run lyx creel read` wording; no board entry owns it.

## Decisions

### D1 — Down asks the watchdog to stop when it ended the server

Decision: when `Engine.Down` finds no session left and runs its server teardown, it writes a stop request into the told discover signal file, the file a cold boot already touches to wake the daemon.
It writes the stop whether or not the server then confirms gone.
The request is a fixed marker as the file's content; a cold boot's touch truncates it to empty, so a racing `up` turns a pending stop back into an ordinary discover.
Standalone mode, which tells no discover signal path, writes nothing.
A `down` that leaves sibling sessions writes nothing.

Rationale: every `Engine.Down` caller gets the stop, and the daemon already watches that file, so no new watch or file is added.

### D2 — A stop-woken cycle with an idle listing exits at once

Decision: the daemon reads the discover signal's content before removing it.
A stop marker runs the discovery cycle without the 5-second base-cadence floor, but never sooner than 1 second after the previous cycle.
If that cycle's listing is idle (`sessionsAreIdle`), the daemon exits without waiting for `watchdogHubIdleCycles` idle cycles.
If the listing is affirmative, the cycle proceeds as an ordinary discover and the daemon keeps running.
A stale marker a daemon finds later is harmless for the same reason: it ends nothing unless the listing is idle.

Bound: the stop lets a writer skip the idle-cycle tolerance and shorten the cadence floor to 1 second, within `PATTERN-daemon-wakeups`' one-second floor.
`sessionsAreIdle` counts a list-sessions error as idle, so a transient error on a stop-woken cycle can end a daemon whose hub still has a session; D3a's exit re-check bounds that to one spawn of a successor.
A daemon whose file watch failed to open never sees the stop and exits by the idle-cycle rule as today.

Rationale: `down` then returns with the watchdog out of its loop, instead of one lingering through the idle cycles' backed-off cadence.

### D3 — The down verb waits for the daemon to exit, bounded

Decision: `lyx reed down`, in hub mode and after `Engine.Down` succeeds and reports it ended the server, polls the watchdog's single-instance lock until it is free, for at most 10 seconds.
A free probe releases the lock at once.
On timeout it logs one Warn naming the hub and the lock path and still succeeds; its envelope is unchanged.
`Engine.Down`'s result reports whether it ended the server.
The bound and the poll interval are injectable, so a test never waits them out.

Bound: the probe holds the lock only between its acquire and its release; a daemon spawned in that instant loses the race and exits, and the hub's next `up`, `attach` or `resume` spawns one.
A free lock means the daemon has left its loop and released the lock; the exiting process may still be running D3a's one re-check listing and at most one successor spawn when `down` returns.

Rationale: #679 expects no reed watchdog after `down`; a free lock is the daemon's own proof it has left its loop, and the process that remains ends after D3a's one re-check.
`down` stays idempotent and never fails over the daemon.

### D3a — An exiting daemon re-checks for sessions after releasing its lock

Decision: when the daemon leaves its loop by D2's stop, by the idle-cycle rule or by D4's idle retire, it releases its lock and lists the hub's sessions once more.
An affirmative listing spawns a successor through `reedengine.SpawnWatchdog`.
A cancelled context spawns nothing.

Bound: one extra list-sessions and at most one spawn per daemon exit; the successor is single-instance by the lock.

Rationale: closes the window where a session boots while the daemon is exiting (it holds the lock through its in-flight reaps), so the boot's own spawn loses the lock race and the hub is left with a session and no watchdog.

### D4 — A replaced binary respawns the daemon only for a live hub

Decision: the binary-replaced check moves after the cycle's session listing.
A replaced binary with an affirmative listing retires the daemon and spawns a successor, as today.
A replaced binary with an idle listing ends the daemon with no successor of its own; D3a's re-check still runs.

Bound: a successor is spawned whenever the cycle's listing or D3a's re-check finds a session.

Rationale: the issue's respawn onto the redeployed dev binary came from an idle daemon retiring into a fresh one that had nothing to watch.

### D5 — The Claude engine's env file re-prepends the spawning binary's directory

Decision: `bash-env.sh` keeps `exec </dev/null` and gains a statement that, when `LYX_BIN` is set, prepends `LYX_BIN`'s directory to `PATH`, quoting the value so a path with spaces holds.
It reads `LYX_BIN` when each Bash command runs, so a resumed session follows the binary its pane's latest prelude exported.
On Windows, where the Bash tool is Git Bash and `LYX_BIN` is a Windows path, the directory is converted with `cygpath -u`, and the statement does nothing when `cygpath` is absent.
With `LYX_BIN` unset the statement does nothing.
The content stays a per-OS constant in the Claude engine.

Bound: the statement prepends only the parent directory of `LYX_BIN`, the same directory reed's pane prelude already puts first in the pane shell, so the Bash tool shadows exactly the executables the pane shell already shadows and no others.
It adds no directory when `LYX_BIN` is unset or, on Windows, when `cygpath` is absent.

Rationale: Claude Code runs the env file in the Bash tool's shell before every command, after the shell snapshot that re-ran the operator's rc, so the prepend wins there; every lyx agent is a Claude session spawned through this engine and runs bare `lyx`.

### D6 — `LYX_BIN`'s key is declared beside the other strand env keys

Decision: the `LYX_BIN` key's declaration moves from `internal/reedengine`'s unexported constant to an exported one in `internal/agentname`, beside `StrandNameEnv` and `ParentEnv`, and both reed's prelude and the Claude engine read it from there.

Rationale: `agentname` already declares the env keys reed's launch script exports, and the Claude engine already imports it; a binary path stretches the package's naming role, accepted to keep one declaration of every launch-script key.

### D7 — The pane shell's own resolution stays as built, now documented

Decision: reed's prelude is unchanged: it is sourced in the pane shell after that shell's rc, so the pane shell resolves bare `lyx` to the spawning binary.
`internal/reedengine/doc.go` states that a fresh shell a strand's command starts re-runs its rc and can reorder PATH, that `LYX_BIN` names the spawning binary there, and that a shuttle engine re-asserts `LYX_BIN`'s directory in its agent's tool shell.

Rationale: the defect reproduces only in shells started below the pane shell, which the prelude cannot reach.

### D8 — The sandbox suites name where the check holds

Decision: each suite's `command -v lyx` pre-condition bullet names the shell it runs in; no other suite line changes, except that the creel suite's closing sentence after its pre-condition bullets counts only the checks that remain:

- reed suite: the pane shell of a strand whose command does not start a fresh shell, with a pointer to the Scope exclusion for strands that do.
- creel suite: its interactive strands, the plain `--cmd claude` and `--cmd bash` strands, start fresh shells, so its `command -v lyx` bullet is reworded to say the check does not hold in those strands, with a pointer to the Scope exclusion and to `SANDBOX-SHUTTLE-SUITE.md` for the agent Bash tool check; its `LYX_BIN`-is-set bullet stays as written.
- shuttle suite: the spawned agent's Bash tool, where D5 makes it hold.

Rationale: the issue reported the suite pre-condition as failing; the reed and shuttle checks now run where the guarantee holds, and the creel suite, whose interactive strands are all outside the guarantee, names that instead of checking it.

## Constraints

- `PATTERN-told-geometry`: `Engine.Down` writes only the told discover signal path and learns nothing of the daemon's lock.
- `PATTERN-shuttle-provider-seam`: the `CLAUDE_ENV_FILE` change lives only under `internal/shuttleengine/claudeengine`; reed stays engine-neutral.
- `PATTERN-leaf-packages`: the new `agentname` constant adds no import to that leaf.
- `PATTERN-shell-mechanics-seam` does not bind the env file: it is a script Claude Code sources, not a pane-shell command string.
- `PATTERN-daemon-wakeups`: stop wakes keep a 1-second floor (D2); `down`'s lock wait is bounded (D3).
- `PATTERN-log-levels`: the lock-wait poll logs nothing per poll; only its timeout logs.
- `PATTERN-test-isolation`: a live-tmux case runs through `tmuxkit.Main`.

## Auto-mode assumptions

- "Ends the watchdog of that worktree reed session" in #679 means the hub watchdog ends once the hub has no session left; ending it while siblings are live would drop their name repair.
- Claude Code runs `CLAUDE_ENV_FILE` after its shell snapshot, in a fresh shell per command, so a PATH set there wins and does not accumulate.
- The re-test's failing pane was a shell started below the pane shell: a `--cmd bash` strand or a Claude strand's Bash tool.

## Open risks

- If a Claude Code release runs `CLAUDE_ENV_FILE` before the snapshot, D5 stops winning; the shuttle suite's pre-condition check is the live detector.
- A daemon with an in-flight orphan reap waits for it before exiting, so a `down` that lands during a reap can hit the 10-second bound and warn.
- `creel-stage1-fixes` edits other lines of `SANDBOX-CREEL-SUITE.md`; this task changes only its pre-condition bullet, and whichever task lands second merges the other's lines.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - d8-creel-check-no-strand (class design, severity MEDIUM, open in rounds 1, 2)
  - free-lock-vs-exiting-daemon (class consistency, severity MEDIUM, open in rounds 1, 2)
<!-- /lyx:carry-over Discussion-Review -->

<!-- lyx:carry-over Plan-Review -->
- Plan-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/plan/round-2-review.md; fixer report: _lyx/reviews/plan/round-2-fixer-report.md.
  - watchdog-down-poll-unannotated (class scope, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Plan-Review -->

## Acceptance criteria

- `lyx reed down` of a hub's last session returns with the watchdog lock free, and the exiting daemon process ends after its post-release re-check without spawning a successor.
- A `down` whose server teardown does not confirm gone still writes the stop marker, returns its error and waits for nothing.
- A daemon leaving its loop on a cancelled context spawns no successor whatever its post-release listing.
- `lyx reed down` of a session with a sibling session live leaves the watchdog running and serving the sibling.
- A stop-woken cycle with an idle listing exits the loop at once; with an affirmative listing it proceeds and the daemon keeps running.
- A stop-woken cycle runs no sooner than 1 second after the previous cycle.
- A cold-boot touch after a stop and before the daemon consumes it leaves the daemon running for the new session.
- A stale stop marker found by a daemon whose hub has a session ends nothing.
- An exiting daemon whose post-release listing is affirmative spawns one successor; one whose listing is idle spawns none.
- A replaced binary with an idle listing ends the daemon with no successor of its own; with an affirmative listing it spawns a successor.
- `down`'s lock wait that times out logs one Warn naming the hub and lock path, and `down` still succeeds with an unchanged envelope; a `down` during an in-flight reap still succeeds.
- Standalone `down` writes no signal and waits for nothing.
- A bash shell whose rc prepends a directory holding a decoy `lyx`, then runs the env file with `LYX_BIN` set to a path containing a space, resolves `command -v lyx` to `LYX_BIN`.
- The env file with `LYX_BIN` unset leaves `PATH` unchanged and still makes `/dev/null` the default stdin.
- In the shuttle sandbox suite, `command -v lyx` from a spawned agent's Bash tool names `LYX_BIN`, on a machine whose rc prepends another `lyx`.
- Reed's prelude and the Claude engine read the same exported `LYX_BIN` key.

## Notes for the plan writer

- The watchdog loop already has test seams for timing, the file-watch opener, the replaced check and the respawn (`c.respawnWatchdog`); the stop, re-check and replaced-ordering cases fit there.
- `consumeDiscoverSignal` removes the file without reading it today; D2 needs read-then-remove.
- The Claude engine's `doc.go` says the env file "runs that one statement and nothing else"; that sentence changes with D5.
