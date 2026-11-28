loom halted failed at Code-Burler-Review

```
shedadapters: Code-Burler-Review (burler-review): round 1 attempt 1: review: burler: fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYMHUB-missing-origin-unreachable/b61ee992-0917-48af-b208-9de75dedf8ee/subagents/agent-aec6cd6f2c4e8dfd2.jsonl" ran a git-mutating command ("S=/home/knatte/Code/loomyard-LYMHUB/missing-origin-unreachable/.scratch/exp2; mkdir -p $S; git init -q $S/r; git -C $S/r commit -q --allow-empty -m c; ln -s /nonexistent-mount/x.git $S/link.git; git -C $S/r remote add origin $S/link.git; git -C $S/r push origin HEAD 2>&1 | head -1; echo \"--- -C relative\"; git -C $S/r remote set-url origin ../gone.git; env -C $S git -C r ls-remote origin 2>&1 | head -1; ls -d $S/gone.git $S/r/../gone.git 2>&1") — a fork must never run a state-mutating git command outside a repository it created
```

producer: Code-Burler-Review
state: failed
anomaly: producer-hard-failure
history_entries: 8
trace_file: /home/knatte/Code/loomyard-LYMHUB/missing-origin-unreachable/.lym/logs/trace-20261128T102442Z-7723990c1c822b39-784009.log

history:
- Preflight / done / 2026-11-28T09:59:03Z
- Record-Write / done / 2026-11-28T09:59:04Z
- Drift-Baseline / skipped / 2026-11-28T09:59:05Z
- Drift-Await / skipped / 2026-11-28T09:59:05Z
- Drift-Expect / skipped / 2026-11-28T09:59:05Z
- Darn-Write / done / 2026-11-28T10:24:05Z
- Drift-Verify / skipped / 2026-11-28T10:24:07Z
- Code-Bouncer / stuck / 2026-11-28T10:24:41Z / routed to Code-Burler-Review
