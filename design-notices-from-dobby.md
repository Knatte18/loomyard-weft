# Dobby sends the batten notices, not a row of the batten recipe

`notices-from-dobby` · Note · enhancement, dobby, batten

## Seen

The quiet, driver-dead, api-error and usage-limit notices are built in `internal/battenshed/notice.go`, as a row of each batten run's own recipe, and run inside the pass dobby spawns for that run (operator, 2026-10-29).
Dobby is the common driver of every batten run under its beam, the pawl run of a beam included, which the dobby of the beam's parent drives.
Its loop already reads every run's state, writes a heartbeat and posts its own notices, so it sees what each notice needs.

## Idea

Dobby detects and sends these notices itself, in its loop over the runs it drives, and the batten recipe keeps no notice row.
A run waiting on a person, such as a pull request, is then read by the same loop through the wait markers it already honours, so a long wait never reads as a stall.
The detection logic, its keys and its idle clocks move as they are; only the place that runs them changes.

## Open

- What replaces the notice row's place in the batten pass, and how a notice's key and stamp survive a dobby re-exec.
- Whether the notices of a run driven by another machine's dobby stay with that dobby's machine.
- The order against `pr-review-row`, which registers its wait marker for the existing quiet check first.
