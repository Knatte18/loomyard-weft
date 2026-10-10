# Reed watchdog children still burn idle CPU

`watchdog-child-cpu` · Note · enhancement, reed

From `daemon-wakeups` (2026-10-10, `docs/benchmarks/daemon-idle-cpu.md`): after the run, `lyx reed watchdog` spawns `list-sessions` sevenfold less often (11.8 to 1.6 per minute), but idle CPU stayed at about 100 s per hour, since about 85 s per hour is spent in child processes the benchmark shim cannot see.

Idea: find which processes the watchdog's children are (likely the idle sessions' own watchers) and bring them under `PATTERN-daemon-wakeups`.
