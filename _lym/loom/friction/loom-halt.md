loom halted failed at Plan-Burler-Fix

```
shedadapters: Plan-Burler-Fix (burler-fix): round 1 attempt 1: fix: burler: twill fixer fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-single-repo-namespace-layout/9d6e0839-6d81-4e6a-8279-290caf461285/subagents/agent-a4d99d80e465bce26.jsonl" ran a git-mutating command ("d=/home/knatte/Code/loomyard-LYMHUB/single-repo-namespace-layout/.scratch/f12probe; mkdir -p $d && cd $d && git init -q --bare bare.git && git init -q w && git -C w commit -q --allow-empty -m x && mkdir -p bare.git/refs/lym/heads && touch bare.git/refs/lym/heads/main.lock && git -C w push \"$d/bare.git\" HEAD:refs/lym/heads/main --force-with-lease=refs/lym/heads/main: 2>&1; echo \"exit $?\"; git --version") — a fork must never run a state-mutating git command; way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run
```

producer: Plan-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/single-repo-namespace-layout/.lym/logs/trace-20261121T134336Z-ed5f221df4ec762a-3446284.log

history:
- Preflight / done / 2026-11-19T19:34:51Z
- Motif / skipped / 2026-11-19T19:35:03Z
- Drift-Baseline / skipped / 2026-11-19T19:35:10Z
- Parts-Write / skipped / 2026-11-19T19:35:22Z
- Parts-Plan-Write / skipped / 2026-11-19T19:35:34Z
- Plan-Write / done / 2026-11-19T20:02:17Z
- Plan-Bouncer / stuck / 2026-11-21T13:28:09Z / routed to Plan-Burler-Review
- Plan-Burler-Review / stuck / 2026-11-21T13:43:21Z / routed to Plan-Burler-Fix
