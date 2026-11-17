---
round: 1
focus:
  - "`internal/proc/memorycap_linux.go` `scopeCgroup`: `io.ReadAll` on the start pipe waits for EOF, and the shell closes fd 3 only after `cat`. Settle whether the read deadline and a held-open write end (the scope's own copy, a failed `systemd-run`) can turn a missed or late listing into a wrong `ok` or a stall, and whether an empty or partial listing reads as 'scope never ran the command'."
  - "`killScope` after a kill at the cap: settle whether a failure other than `ErrNotExist` or `ENODEV` (permission on `cgroup.kill`, a cgroup v1 or a delegated tree without the file, `os.WriteFile` with mode 0 and its open flags) fails a run that was otherwise correctly reported as killed, and whether joining that error into `err` changes what callers of `RunMemoryCapped` and the `lym gate redundant` mutant path see."
  - "`OOMPolicy=continue` in `memoryScopeArgs`: settle that a kill at the cap still reaches the command with SIGKILL and `killed` stays true, that a surviving scope process cannot outlive the call on a path that skips `killScope` (the cancel path, a non-SIGKILL exit, the data-limit fallback), and that the cgroup path from `/proc/self/cgroup` is the scope's own and not a parent's."
  - "`memorycap_linux_test.go` new test: settle that it pins behavior the existing capped-run tests in the file do not (the child's leftover process ending), that its 10 s poll cannot pass on a recycled pid or a zombie `/proc` entry, and that it is not redundant with another test; the redundancy report lists it as not judged."
  - "Docs and comments in `doc.go`, `memorycap.go` and `memorycap_linux.go`: settle that each claim (leftover processes killed, no oom-kill result, KDE notifier) matches what the code does and that the package docs the repository's rules name for a behavior change landed in the same diff."
---

Round 1 seed.
The decision record holds only the board entry under `## Goal`, with no added entries, so the entry is the measuring stick.
The entry asks that capped scopes stop raising the desktop low-memory notice while the cap's tests keep working.
The diff takes the `OOMPolicy=continue` route and adds a `cgroup.kill` cleanup; the focus entries target that cleanup's failure paths.
