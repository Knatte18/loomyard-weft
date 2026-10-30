# Dobby sends the run notices, not a row of the batten recipe

`notices-from-dobby` · Task · enhancement, dobby, batten · running · Describe

## Seen

The quiet, driver-dead, api-error and usage-limit notices are built in `internal/battenshed/notice.go`, as a row of each batten run's own recipe, and run inside the pass dobby spawns for that run (operator, 2026-10-29).
They go out as `[batten notice]` lines from `ly:BATTEN`, and the orch role, the stencils and the hub's rules read that line.
Dobby is the daemon that keeps track of the runs and the common driver of every one under its beam, the pawl run of a beam included, which the dobby of the beam's parent drives.
Batten is only a profile of a shed, and a shed drives nothing itself: it is driven by dobby.
Dobby's loop already reads every run's state, writes a heartbeat and posts its own notices, so it sees what each notice needs.

## Design (operator, 2026-10-29 and 2026-10-30)

Dobby detects and sends these notices itself, in its loop over the runs it drives, and the batten recipe keeps no notice row.
A run waiting on a person, such as a pull request, is read by the same loop through the wait markers it already honours, so a long wait never reads as a stall.
The detection logic, its keys and its idle clocks move as they are; only the place that runs them changes.

- The marker file that records the notices already sent, one episode key per line, moves to dobby, and lives under `_lym`, so it survives a dobby re-exec and a machine switch.
  It is the existing marker moved, never a new notification flag; the set of notices dobby looks for stays fixed in the code.
- The notice lines are sent by dobby, from `<shortname>:DOBBY`, and read `[dobby notice]` in place of `[batten notice]`.
  The orch role's Notices section, the stencils and the hub's rules that name the old line change in the same task.
- The notices of a run stay with the dobby of the machine that drives it; the orch is not the same on another machine either.
