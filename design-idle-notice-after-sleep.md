# Batten idle notices count machine sleep as idle time

`idle-notice-after-sleep` · Note · bug, batten

## Seen

The machine hibernated from 2026-10-08 22:17 to 2026-10-10 06:01 CEST with three runs in flight.
Minutes after waking, batten raised an idle notice for each run ("agents idle for 31h46m"), although every agent had resumed its turn and was working.
The orch has to check the system log for a sleep window before reading any idle notice (orch role, Investigating a stop).

## Idea

The idle clock measures time the machine was awake: it uses a clock that stops during suspend (`CLOCK_MONOTONIC` on Linux rather than wall time), or drops a gap that matches a sleep window.
After a wake, a run is idle only once its agents have stayed quiet for the idle threshold of awake time.
