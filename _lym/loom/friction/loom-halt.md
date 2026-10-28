loom halted failed at Discussion-Burler-Review

```
shedadapters: Discussion-Burler-Review (burler-review): round 2 attempt 2: review: burler: fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-validator-checks/c03521d9-e53e-4879-808c-02111ba2003a/subagents/agent-af48b9e591d6a4e6f.jsonl" ran a git-mutating command ("git branch -a --contains 71a46b11 2>/dev/null | head; git merge-base --is-ancestor 71a46b11 HEAD && echo IN_HEAD || echo NOT_IN_HEAD; git show --stat 71a46b11 | grep -E \"landingdeps_test|^Date\" ") — a fork must never run a state-mutating git command
```

producer: Discussion-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 7
trace_file: /home/knatte/Code/loomyard-LYMHUB/validator-checks/.lym/logs/trace-20261028T105548Z-7bfaff2abe252dc3-993265.log

history:
- Preflight / done / 2026-10-28T10:30:30Z
- Loom-Preflight / done / 2026-10-28T10:30:30Z
- Discussion-Write / done / 2026-10-28T10:43:12Z
- Discussion-Bouncer / stuck / 2026-10-28T10:43:53Z
- Discussion-Burler-Review / stuck / 2026-10-28T10:51:23Z
- Discussion-Burler-Fix / stuck / 2026-10-28T10:55:05Z
- Discussion-Bouncer / stuck / 2026-10-28T10:55:48Z
