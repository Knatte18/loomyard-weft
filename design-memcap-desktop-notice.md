# A capped test run raises a desktop low-memory notice

`memcap-desktop-notice` · Task · bug, gate, process · running · Describe

## Seen (2026-11-17, Neve, KDE Plasma)

`redundant-mutant-memory-cap` runs each test binary under `systemd-run --user --scope -p MemoryMax=<cap>`.
The scope lands in `app.slice`, and when the kernel ends a run at its cap,
the desktop shows "The background service run-p<pid>-i<id>.scope has been terminated by the Linux kernel because the system is low on memory".
The machine is not low on memory: the kill is the scope's own memcg (`CONSTRAINT_MEMCG`), with about 19 GB available.

Each gate run of the cap's own tests (`proc.test`, `testcover.test`, the `mutants.test` fixtures) raises several such notices,
14 in five minutes while that run's reviews ran,
and every capped mutant of a real `lym gate redundant` run will raise one more.
The operator reads them as a machine about to freeze again.

## Idea

- Start the scopes in a slice of lym's own (`systemd-run --slice=lym-tests.slice`, or a name per hub), not `app.slice`,
  and check whether the desktop's OOM notifier still reports kills there; it may watch only `app.slice` units.
- If it still does, find what the notifier keys on (the unit's result `oom-kill`, its slice, or the kernel event),
  and choose a cap path that ends the run without it, such as `MemoryHigh` plus lym's own kill, or the `RLIMIT_AS` path.
- The tests that prove the cap keep working on Neve without one desktop notice per run.
