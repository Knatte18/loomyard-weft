loom halted failed at Webster

```
loop-exited: loop b0fbebfd9f0f68de ended without an envelope; its output is in /home/knatte/Code/loomyard-LYMHUB/claude-accounts-per-worktree/.lym/shed/claude-accounts-per-worktree/steps/loop.log; step Drift-Await was in flight under trace e2126fe17e0cfddd, and the status file now reads Webster; loop exit: not recorded, since no process marked the loop's exit, as after a kill of the loop and its keeper that no process can catch or a stop of the machine; step stderr: time=2026-11-18T13:56:00.081+01:00 level=WARN msg="shuttle: session state disagrees" trace=e2126fe17e0cfddd strandGUID=5cbffc253a5bb4f6a5785956571107fa loop=held state=busy cause=background stands=loop | time=2026-11-18T14:03:13.420+01:00 level=WARN msg="shuttle: session state disagrees" trace=e2126fe17e0cfddd strandGUID=5cbffc253a5bb4f6a5785956571107fa loop=held state=busy cause=background stands=loop | time=2026-11-18T14:11:44.054+01:00 level=WARN msg="shuttle: session state disagrees" trace=e2126fe17e0cfddd strandGUID=5cbffc253a5bb4f6a5785956571107fa loop=held state=busy cause=background stands=loop; stopped shuttle runs: ly:claude-accounts-per-worktree:webster
status error: shedverbs: step Webster interrupted (loop-exited); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lym shed step claude-accounts-per-worktree --until-stop
```

producer: Webster
state: failed
anomaly: interrupted-stop
history_entries: 19
trace_file: /home/knatte/Code/loomyard-LYMHUB/claude-accounts-per-worktree/.lym/logs/trace-20261118T123253Z-e2126fe17e0cfddd-3816240.log

history:
- Preflight / done / 2026-11-18T11:16:59Z
- Motif / skipped / 2026-11-18T11:17:05Z
- Drift-Baseline / skipped / 2026-11-18T11:17:15Z
- Parts-Write / skipped / 2026-11-18T11:17:20Z
- Parts-Plan-Write / skipped / 2026-11-18T11:17:34Z
- Plan-Write / done / 2026-11-18T11:46:51Z
- Plan-Bouncer / stuck / 2026-11-18T11:47:25Z
- Plan-Burler-Review / stuck / 2026-11-18T11:59:03Z
- Plan-Burler-Fix / stuck / 2026-11-18T12:07:44Z
- Plan-Bouncer / stuck / 2026-11-18T12:08:20Z
- Plan-Burler-Review / stuck / 2026-11-18T12:16:20Z
- Plan-Burler-Fix / stuck / 2026-11-18T12:22:30Z
- Plan-Bouncer / stuck / 2026-11-18T12:23:01Z
- Plan-Burler-Review / stuck / 2026-11-18T12:27:00Z
- Plan-Burler-Fix / stuck / 2026-11-18T12:30:54Z
- Plan-Bouncer / awaiting / 2026-11-18T12:31:32Z
- Plan-Bouncer / done / 2026-11-18T12:32:43Z
- Drift-Await / skipped / 2026-11-18T12:32:53Z
- Drift-Expect / skipped / 2026-11-18T12:32:58Z
