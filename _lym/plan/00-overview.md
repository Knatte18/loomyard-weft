---
format: 5
approved: true
language: go
---

# Plan: dobby and adopt hardening

A dobby survives every deploy by re-exec and never ends silently, logs only its changes plus a ten-minute heartbeat in a bounded `dobby.log`, and counts and tidies only the landings its own machine made.
The adopt machine check runs under its own gate key `adopt` through a hidden `lym adopt check` that reports each tier and its end to the orch and shows its progress in `lym beam status`.
The cards run in order: the landing machine on the shed's history and its count (1 to 4), the dobby's re-exec, log and exits (5 to 10), then the adopt gate key, the in-process gate, the check verb, its spawn and its status line (11 to 15).

## Card Index

1 — shed-history-machine — stamp a told machine on each shed history entry
2 — batten-tells-machine — tell batten's shed this machine's name and refuse a run with none
3 — landings-by-machine — count only the landings this machine made
4 — landings-flag — empty a pre-machine landing list once before the first count
5 — selfexec-callbacks — tell a selfexec caller before an exec and after a failed one
6 — dobby-log-file — append each dobby log line by itself, rotate at 1 MiB, raw output to dobby.out
7 — dobby-start-and-exits — take the lock first, wait longer after a re-exec, log every way a dobby ends
8 — dobby-worktree-gone — end a beam dobby whose worktree is gone and stop its tidy agent
9 — dobby-change-log-heartbeat — log only dobby's changes and a ten-minute heartbeat
10 — dobby-cli-exits-and-out — trace each exit before Run and move raw output onto dobby.out
11 — gate-adopt-key — add the adopt gate key and seed it from a tuned finalize
12 — gate-run-in-process — run a gate key in-process with a per-tier observer
13 — adopt-check-verb — add the hidden adopt check that runs the adopt gate and reports per tier
14 — adopt-spawns-check — start the adopt check from lym adopt and skip a live one
15 — beam-status-adopt — show the adopt check in lym beam status

## Shared Decisions

- The ten-second re-exec lock wait, the 1 MiB rotation bound and the ten-minute heartbeat are package constants (`reexecLockWait`, `logRotateBytes`, `heartbeatInterval`), per the record's auto-mode assumptions; `Deps.ReexecLockWait` exists only so tests shorten the wait.
- Only the holder of a file's dobby lock rotates `dobby.log` and `dobby.out`; every other writer appends through `dobbyengine.AppendLog` with `rotate` false, and the one exception is the Windows spawner's rotation of `dobby.out` behind a free-lock read.
- No append of a dobby line creates a directory, so the worktree-gone exit writes nothing at the removed worktree's path.
- The adopt check's files (`check.json`, `check.out`, `check.lock`) sit beside `suite.log` under `fabricengine.AdoptScratchDir` and are named only in `internal/adoptcli`.
