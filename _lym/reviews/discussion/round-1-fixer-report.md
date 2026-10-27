# Fixer report — discussion, round 1

Every finding is fixed in `_lym/discussion/decision-record.md`, and the round is recorded in `_lym/discussion/support-log.md`'s Review rounds.

## Changes

- F1, fixed as proposed: Decision 3 lists the two exits before `Run`.
  A `hubDobbyDeps` wiring failure writes its line to `Geometry.LogPath` directly and posts the failed-re-exec notice after a re-exec.
  A `placeDobby` refusal leaves only its `dobby.out` envelope, stated as the bound.
  The Acceptance criterion on Decision 3's exit paths now names the wiring failure.
  The self-check refined this further: the notice route, its worktree condition, and `lym beam start` in the notice.
- F2, fixed as proposed: Decision 10 adds a non-blocking lock beside `check.json`.
  The verb, not the spawner, truncates `suite.log` and writes the fresh running record once it holds the lock.
  A check that finds the lock held exits 0 and writes nothing.
  `lym adopt` spawns none while a live check runs.
  The verb resolves the prime from the hub whatever its cwd.
  The Bound no longer claims the gate slots serialize checks; the hub holds `slots: 3`, so the text names no count.
  Criteria were added for a second check while one runs and for `lym adopt` with a live check.
- F3, fixed differently: the proposal named `hubreconcile.EnsureOrWarn`.
  The self-check (fix-proposal) showed that `EnsureOrWarn` goes on after a failed reconcile (`internal/hubreconcile/hubreconcile.go:59-63`), which leaves F3's green-on-tier-0 case open.
  Decision 10 therefore calls `hubreconcile.Ensure`, and a failed reconcile ends the check `failed` with the reconcile's error and way forward.
  The migration criterion covers a stale stamp and a failed reconcile.
- F4, fixed as proposed: `check.json` carries `proc.StartTime` start ticks.
  Decision 11 reads a running record as ended when its pid is dead or its start ticks differ, and the beam status criterion was extended to match.
- F5, fixed as proposed: Decision 2's re-exec test is pid plus start ticks, and the state record gains the field, written by `recordStart`.
  The Bound gives both orderings with a second dobby, and a criterion was added for each case.
  The self-check added that a record with this pid and no start ticks reads as a re-exec.
- F6, fixed as proposed: Decision 6 logs the tick-failure and owner-read episodes, and they were added to the logged-events criterion.
- F7, fixed as proposed: Decision 5 says only the lock holder rotates; a criterion was added.
- F8, fixed as proposed: Decision 7 refuses the batten run when `machinename.Name` fails; added to the history-entry criterion.
- F9, fixed as proposed, taking the bound option rather than moving the wiring: Decision 1's Bound states the unregistered window and the teardown escape.
  The self-check (fix-claims) showed that the proposal's "exits at its first probe tick, which finds no worktree" is false.
  No exit for a gone worktree exists (`internal/dobbyengine/loop.go:212-224`).
  So Decision 1 now adds that exit, Decision 3 lists its line, and a criterion covers it.
- F10, fixed as proposed: two criteria were added, one for the re-exec's first tick and start heartbeat and one for a fresh record and emptied log.
- F11, fixed as proposed: the Scope Out bullet drops the `test-logs-leak-into-bolt` clause.
- F12, fixed as proposed: Decision 4 says any start keeps `State.Tidy`.
- F13, fixed as proposed: Decision 9 reads "whenever the file holds no `adopt`", and the second Open risks line says to set `adopt` through `lym config gate`, never by deleting it.
- F14, fixed as proposed: Decision 8's Bound says only `settleTidy` clears `LandedSinceTidy`.
- F15, fixed as proposed: the last Constraints bullet is gone, and the PATTERN-test-speed bullet names `internal/selfexec`'s refusal.
  The PATTERN-durable-vs-ephemeral-state and PATTERN-tolerant-records bullets also name the check's lock file and the start-ticks field.

## Self-check

- fix-consistency, MEDIUM: Decision 5's heading, its first line and the Scope In Part 2 bullet said only the seam writes `dobby.log`, against the wiring-failure line. Fixed as proposed: they now say `dobby.log` holds only the dobby's own lines.
- fix-consistency, LOW: Decision 3's heading overclaimed against the `placeDobby` bound. Fixed as proposed: "Every way a placed dobby ends".
- fix-consistency, LOW: the first Open risks line was stale against the same bound. Fixed as proposed.
- fix-consistency, LOW: Decision 3's first paragraph and the support log's Question ledger missed the wiring-failure notice. Fixed as proposed.
- fix-claims, MEDIUM: Decision 1's teardown-escape claim does not hold in code. Fixed with an own change: an exit for a gone worktree was added (see F9).
- fix-claims LOW, merged with fix-proposal MEDIUM: a pre-change state record has no start ticks, so the first re-exec reads as fresh. Fixed as proposed: an empty start-ticks value with a matching pid reads as a re-exec, with a criterion row.
- fix-proposal, MEDIUM: `EnsureOrWarn` lets a failed reconcile run the template's tiers green. Fixed as proposed: `Ensure`, and the check fails (see F3).
- fix-proposal LOW, merged with fix-way-forward MEDIUM: the wiring-failure notice had no route and needs a resolved worktree. Fixed as proposed: it posts through `orchcli.NotifyFromScoped` over a worktree `runDobby` resolves itself, and only the log line remains when that resolution failed.
- fix-way-forward, LOW: the wiring-failure notice named no restart. Fixed as proposed: it names `lym beam start` once the cause is fixed.
- fix-proposal LOW, merged with fix-way-forward LOW: a second `lym adopt` inside the spawn-to-lock window can name a pid that never ran. Fixed as proposed: Decision 10's Bound states it.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
