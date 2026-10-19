loom halted failed at Plan-Burler-Review

```
shedadapters: Plan-Burler-Review (burler-review): round 1 attempt 1: review: burler: shuttle run: seatengine: chair wait: shuttle: reed did not track strand "7ddd6510d449416ce21003067960ea12" on 2 consecutive liveness checks: reed's strand table no longer holds this run's strand — reed's bookkeeping was reset under the run (a reed remove/down, a lost or rebuilt reed.json, or a worktree renamed while the run was in flight), which says nothing about the agent: its process may still be working in its pane. Check "lyx reed status"
```

producer: Plan-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 31
trace_file: /home/knatte/Code/loomyard-LYXHUB/ubuntu-2404-support/.lyx/logs/trace-20261019T180138Z-2fc93e6048e3e543-647681.log

history:
- Preflight / done / 2026-10-19T14:47:28Z
- Loom-Preflight / done / 2026-10-19T14:47:28Z
- Discussion-Write / done / 2026-10-19T14:59:36Z
- Discussion-Bouncer / stuck / 2026-10-19T14:59:57Z
- Discussion-Burler-Review / stuck / 2026-10-19T15:42:14Z
- Discussion-Burler-Fix / stuck / 2026-10-19T15:48:07Z
- Discussion-Bouncer / stuck / 2026-10-19T15:48:34Z
- Discussion-Burler-Review / stuck / 2026-10-19T15:52:42Z
- Discussion-Burler-Fix / stuck / 2026-10-19T15:55:04Z
- Discussion-Bouncer / done / 2026-10-19T15:55:28Z
- Plan-Write / done / 2026-10-19T16:15:45Z
- Plan-Bouncer / stuck / 2026-10-19T16:16:08Z
- Plan-Burler-Review / stuck / 2026-10-19T16:30:53Z
- Plan-Burler-Fix / stuck / 2026-10-19T16:37:05Z
- Plan-Bouncer / stuck / 2026-10-19T16:37:42Z
- Plan-Burler-Review / stuck / 2026-10-19T16:49:47Z
- Plan-Burler-Fix / stuck / 2026-10-19T16:53:02Z
- Plan-Bouncer / stuck / 2026-10-19T16:53:29Z
- Plan-Burler-Review / stuck / 2026-10-19T17:02:04Z
- Plan-Burler-Fix / stuck / 2026-10-19T17:08:39Z
- Plan-Bouncer / awaiting / 2026-10-19T17:09:17Z
- Plan-Bouncer / done / 2026-10-19T17:13:26Z
- Webster / stuck / 2026-10-19T17:19:56Z
- Plan-Write / goto / 2026-10-19T17:25:32Z
- Plan-Write / done / 2026-10-19T17:31:32Z
- Plan-Bouncer / stuck / 2026-10-19T17:32:00Z
- Plan-Burler-Review / stuck / 2026-10-19T17:45:31Z
- Plan-Burler-Fix / stuck / 2026-10-19T17:47:46Z
- Plan-Bouncer / stuck / 2026-10-19T17:54:49Z
- Plan-Bouncer / goto / 2026-10-19T17:55:13Z
- Plan-Bouncer / stuck / 2026-10-19T17:55:22Z
