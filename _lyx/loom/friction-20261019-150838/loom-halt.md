loom halted failed at Discussion-Burler-Review

```
shedadapters: Discussion-Burler-Review (burler-review): round 1 attempt 1: review: burler: fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-ubuntu-2404-support/2a2c97bd-9cec-4552-b17e-bba949d7358e/subagents/agent-a2dfed49a3f9fd78e.jsonl" ran a git-mutating command ("git help -m cherry-pick 2>/dev/null | col -b | grep -n -A14 \"^ *--empty\" | head -30") — a fork must never run a state-mutating git command
```

producer: Discussion-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 4
trace_file: /home/knatte/Code/loomyard-LYXHUB/ubuntu-2404-support/.lyx/logs/trace-20261019T145957Z-a42b40cb49e84e85-3297421.log

history:
- Preflight / done / 2026-10-19T14:47:28Z
- Loom-Preflight / done / 2026-10-19T14:47:28Z
- Discussion-Write / done / 2026-10-19T14:59:36Z
- Discussion-Bouncer / stuck / 2026-10-19T14:59:57Z
