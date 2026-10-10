# Daemon wakeups: fswatch leaf, reed wake cadence, orch held-injection log

Batches 01 (fswatch-leaf) and 05 (reed-wake-cadence) were recorded `done` before this session resumed.
Batch 09 (orch-held-injection-log) had failed earlier, was reset to its start by `ly:orch`, re-begun and re-forked here.
Its digest reports `status: done` at head `a353b4d023a5bfab3c10b1b3753ebabcad9f5de3`.

The batch 09 prompt file carried no sandbox-hub rule and Master cannot edit it.
`ly:orch` therefore had Master relay the rule to the fork by message while it ran: build a sandbox hub under the scratch dir from local bare repos, and never run `lyx fabric clone/add/remove`, push, or read the real hub's remotes.

## What batch 09 built

Per the fork's reply, batch 09 committed cards 9 to 14:

- Card 9: the orch watcher records and logs why each injection waited.
- Card 10: the shell seam gains `ChangeDir`.
- Cards 11 and 12: the resume-context engine and the hidden `lyx orch resume-context` verb.
- Card 13: `PATTERN-daemon-wakeups` and its guard.
- Card 14: the idle-CPU benchmark report under `docs/benchmarks/`.

The fork reports `go build ./...`, `go test ./...`, `go test -tags integration ./...` and `lyx loom lint-comments` green.
Its benchmark found no change in watchdog idle CPU, about 100 s per hour before and 102 s after, with `list-sessions` spawns down from 59 to 8 per five minutes.
It was run against a sandbox hub.

## Deviations

- New files `internal/orchengine/resumecontext.go` and `internal/orchcli/resumecontext.go`.
- `internal/orchcli/refusal_integration_test.go` gained `resume-context` in the prime-only refusal loop.
- The record-batch envelope also listed informational scope-outside-plan warnings for symbols in `cmd/lyx/daemonwakeups_test.go`, `internal/orchcli`, `internal/orchengine` and `internal/shell`.

`PATTERN.md` is within a few bytes of its size budget, so the next entry needs room freed first.


## Background shells waited out

Master's turn end was counted while these background shells were still running, which comes after `background_shell_wait_min` for a shell only the transcript reports and at once for one the Stop payload reports when Master's output files exist; they may still be running in the session.

- `S=/tmp/claude-1000/-home-knatte-Code-loomyard-LYXHUB-daemon-wakeups/51aa2a9c-200a-4737-96be-412c05352663/scratchpad; cat $S/after.result; chmod +x $S/bench2.sh $S/tmuxcount.sh; $S/bench2.sh before-count $S/bin/lyx-before 300; $S/bench2.sh after-count $S/bin/lyx-after 300; for l in before-count after-count; do echo $l; awk '{print $2}' $S/$l.tmux | sort | uniq -c; done`
