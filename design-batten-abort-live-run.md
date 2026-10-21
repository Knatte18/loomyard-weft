# No way to stop a live run for good: killed parts come back, the claim stays

`batten-abort-live-run` · Note · bug, batten, reed, shed

## Seen (orch, 2026-10-21)

The operator wanted the live loom run `quarry-csharp`, at Discussion-Write, stopped for good and removed.
No verb does that, and the orch's manual stop took several rounds while the run kept coming back:

1. The orch killed the `lyx batten run` and `lyx batten step` processes and the Discussion-Write Claude session.
2. Reed relaunched the strand, since its tmux session still stood, so a new Claude session started.
3. Killing the tmux session on the reed server ended it, but a drive-report writer then started, and loom posted "Run quarry-csharp stopped".
4. `lyx batten status` still read `running` throughout, so the board claim stayed and the entry took no scope edits.
5. `lyx fabric remove` waits up to two minutes for the driver to go quiet, which looked to the operator like a hang.

`lyx batten pause` acts only at the next producer boundary, which a session waiting for its turn never reaches.

## Idea

One verb, such as `lyx batten abort <slug>`, that ends a live run of any state at once and for good:

- stops the batten and loom drivers, every strand of the run and its reed session, so reed relaunches nothing;
- writes no drive report and posts no stop notice beyond one abort notice;
- marks the batten record aborted and clears the board claim, so the entry takes edits again;
- optionally removes the pair, with `--remove`, without the two-minute wait since nothing is left to go quiet.

Task `batten-cancel-verb` covers a run that stopped before its pair exists; the two may become one verb.
Task `run-process-ownership`, where every process a run spawns has one owner that ends them all, is what this verb would call.
