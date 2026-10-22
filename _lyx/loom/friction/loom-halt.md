loom halted failed at Plan-Burler-Fix

```
shedadapters: Plan-Burler-Fix (burler-fix): round 2 attempt 2: fix: burler: twill fixer fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-quarry-csharp/b6e5eb6b-947d-4971-a983-aeacb6cf986b/subagents/agent-af6ad1463741a3a48.jsonl" ran a git-mutating command ("Q=/home/knatte/Code/quarry-LYXHUB/quarry; sed -n 225,320p $Q/internal/engine/name.go; git -C $Q log -1 --format='%h %d' ; git -C $Q tag --points-at HEAD") — a fork must never run a state-mutating git command; way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run
```

producer: Plan-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 20
trace_file: /home/knatte/Code/loomyard-LYXHUB/quarry-csharp/.lyx/logs/trace-20261022T073531Z-666b0b9127a64d03-2037576.log

history:
- Preflight / done / 2026-10-22T05:59:42Z
- Loom-Preflight / done / 2026-10-22T05:59:42Z
- Discussion-Write / done / 2026-10-22T06:09:00Z
- Discussion-Bouncer / stuck / 2026-10-22T06:09:26Z
- Discussion-Burler-Review / stuck / 2026-10-22T06:15:56Z
- Discussion-Burler-Fix / stuck / 2026-10-22T06:22:18Z
- Discussion-Bouncer / stuck / 2026-10-22T06:22:59Z
- Discussion-Burler-Review / stuck / 2026-10-22T06:29:17Z
- Discussion-Burler-Fix / stuck / 2026-10-22T06:34:25Z
- Discussion-Bouncer / stuck / 2026-10-22T06:35:13Z
- Discussion-Burler-Review / stuck / 2026-10-22T06:40:35Z
- Discussion-Burler-Fix / stuck / 2026-10-22T06:43:53Z
- Discussion-Bouncer / awaiting / 2026-10-22T06:44:33Z
- Discussion-Bouncer / done / 2026-10-22T06:45:49Z
- Plan-Write / done / 2026-10-22T07:06:23Z
- Plan-Bouncer / stuck / 2026-10-22T07:07:00Z
- Plan-Burler-Review / stuck / 2026-10-22T07:17:09Z
- Plan-Burler-Fix / stuck / 2026-10-22T07:24:31Z
- Plan-Bouncer / stuck / 2026-10-22T07:25:00Z
- Plan-Burler-Review / stuck / 2026-10-22T07:35:31Z
