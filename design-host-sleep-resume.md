# Runs survive host hibernation and resume on wake

`host-sleep-resume` · Note · enhancement, shuttle, batten

## Idea (operator, 2026-10-12)

lyx should support the host going into hibernation or suspend while runs are live, and pick the runs up cleanly when the machine wakes.

## Seen (2026-10-12)

- The operator hibernated the PC overnight with `driver-arbiter` in Plan-Write.
  On wake the plan writer's turn had ended with `API Error: Connection lost mid-response` (shuttle state `idle-stalled`, cause `api-error`, detail `server_error`), and the pane showed the turn as having run for 10h 51m.
- Someone typed `fortsett` into the pane to resume it; the driver then spawned a fork to diagnose an interrupted stop and restarted `lyx shed step --until-stop`.
- Batten later sent a `[batten notice]` that agents had been idle for almost 11 hours, counting the sleep as idle time.

## Open

- Whether lyx can detect a wake (a wall-clock jump against a monotonic clock, or a systemd-logind sleep signal) and treat it as a known cause rather than a stall.
- Whether a turn cut by a connection lost across sleep should be resumed by the driver itself, without a typed `fortsett`.
- Whether idle timers, watchdog timeouts and batten notices should exclude time the host slept.
