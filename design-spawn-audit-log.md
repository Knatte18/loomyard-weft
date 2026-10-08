# Spawn audit log: every process lyx starts is recorded and traceable

`spawn-audit-log` · Note · feature, reed, shuttle

## Seen

On 2026-10-08 the operator's machine froze and needed a hard reset at about 20:12 CEST while four runs were in flight.
`sar` shows about 800 process starts per second between 19:50 and 20:00, about 500 more live processes by 20:10,
and the user slice at its 16 GiB `MemoryHigh` with swap full, so reclaim thrashed (8366 major faults/s, 768 MB/s reads).
Only about 15 claude sessions and 9 subagents started in that window, and nothing in lyx's logs says what else started the processes or which run, strand or command owned them.
The operator has seen an earlier storm of 100+ claude panes in reed, and could not trace that one either.

## Idea

Every process lyx starts goes through one recorded launch path that appends an audit line to a hub-wide spawn log:
time, pid, parent pid, the lyx command and trace id that started it, hub, run slug, strand name, executable and argv,
and on exit the duration, exit status, peak RSS and bytes read and written (from `rusage` and `/proc/<pid>/io`).
This covers tmux servers, sessions and panes, claude launches, git, and `go build`/`go test` in verify gates.
A sampler, such as the reed watchdog, also records per-strand process-tree counts and memory every few seconds,
so the state just before a freeze survives a reboot.
`lyx` gains a read verb that summarises the log by minute, run, strand and executable, so a storm can be traced to its source after the fact.
