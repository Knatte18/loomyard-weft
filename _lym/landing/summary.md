# A capped test run no longer raises a desktop low-memory notice

`proc.RunMemoryCapped` runs each test binary in a transient systemd user scope with `MemoryMax=<cap>`.
When the kernel killed a binary at its cap, systemd failed the scope with the result `oom-kill`.
KDE Plasma's `oom-notifier` kded module watches every user `Scope` and `Service` unit over D-Bus,
and it shows "terminated by the Linux kernel because the system is low on memory" for each unit that ends with that result.
So each capped run, in the cap's own tests and in `lym gate redundant`, raised a notice on a host with gigabytes free.

## What changes

- `internal/proc`, Linux scope path:
  - The scope runs with `-p OOMPolicy=continue`.
    The kernel still kills the command at the cap, but systemd no longer fails the unit with `oom-kill`, so the notifier stays quiet.
  - With that policy systemd no longer stops the scope's other processes after a kill.
    The scope's start script now writes its cgroup listing to the start pipe instead of an empty line,
    and after a kill at the cap `runMemoryCapped` writes `1` to that cgroup's `cgroup.kill`, ending whatever the command left there.
    A scope already emptied and removed is no error.
    A failed kill is joined to the returned error.
- `doc.go` and `RunMemoryCapped`'s doc state both points.
- New test `TestRunMemoryCapped_AKillAtTheCapEndsWhatTheChildLeftInItsScope`:
  a capped shell starts a long sleep and becomes a hog past the cap,
  and the sleep must be gone soon after the call reports the kill.
  It runs only where the host can cap a scope.

## Design decisions

- The board entry suggested starting the scopes in a slice of lym's own.
  The notifier never checks the slice: it keys only on the unit's `Result` being `oom-kill`, which the module's strings show.
  A separate slice would therefore not stop the notices, so the scopes stay in `app.slice`.
- On Neve (systemd 259), a capped scope with the default policy logged `Failed with result 'oom-kill'`.
  The same scope with `OOMPolicy=continue` logged the OOM kill and ended with `Result=success`.
- `cgroup.kill` (Linux 5.14+) ends every process in the scope, including ones that left the command's process group.
  A process-group kill could miss those, and it could hit a reused group id once the group's leader is gone.
- `MemoryHigh` plus a kill of lym's own, and the `RLIMIT_DATA` path, were not needed: the existing cap and kill report stay as they were.
