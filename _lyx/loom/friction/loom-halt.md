loom halted failed at Plan-Burler-Fix

```
shedadapters: Plan-Burler-Fix (burler-fix): round 1 attempt 1: fix: burler: twill fixer fork "/home/knatte/.claude/projects/-home-knatte-Code-loomyard-LYXHUB-orch-strand-cycle/781dd9f4-82dc-4cd7-a55e-0a2b53f08417/subagents/agent-aebb1c18b39c3e054.jsonl" ran a git-mutating command ("S=/tmp/claude-1000/-home-knatte-Code-loomyard-LYXHUB-orch-strand-cycle/781dd9f4-82dc-4cd7-a55e-0a2b53f08417/scratchpad/repairchk; mkdir -p $S && cd $S && git init -q main && git -C main commit -q --allow-empty -m i && git -C main worktree add -q ../pair 2>&1; mkdir -p $S/emptygit; printf 'gitdir: %s\\n' \"$S/emptygit\" > $S/pair/.git; git -C $S/main worktree repair $S/pair; echo \"exit=$?\"; cat $S/pair/.git; git -C $S/pair rev-parse --show-toplevel; mv $S/pair/.git $S/pair/gitfile.bak; mkdir $S/pair/.git; git -C $S/main worktree repair $S/pair; echo \"exit=$?\"") — a fork must never run a state-mutating git command; way forward: inspect the target's working-tree changes, revert what the fork wrote, and resume the run
```

producer: Plan-Burler-Fix
state: failed
anomaly: producer-hard-failure
history_entries: 16
trace_file: /home/knatte/Code/loomyard-LYXHUB/orch-strand-cycle/.lyx/logs/trace-20261022T161206Z-9e5fd6612f4d889f-3771021.log

history:
- Preflight / done / 2026-10-22T14:43:03Z
- Loom-Preflight / done / 2026-10-22T14:43:03Z
- Discussion-Write / done / 2026-10-22T14:56:17Z
- Discussion-Bouncer / stuck / 2026-10-22T14:56:39Z
- Discussion-Burler-Review / stuck / 2026-10-22T15:07:18Z
- Discussion-Burler-Fix / stuck / 2026-10-22T15:12:58Z
- Discussion-Bouncer / stuck / 2026-10-22T15:13:29Z
- Discussion-Burler-Review / stuck / 2026-10-22T15:24:35Z
- Discussion-Burler-Fix / stuck / 2026-10-22T15:31:07Z
- Discussion-Bouncer / stuck / 2026-10-22T15:31:41Z
- Discussion-Burler-Review / stuck / 2026-10-22T15:39:52Z
- Discussion-Burler-Fix / stuck / 2026-10-22T15:42:13Z
- Discussion-Bouncer / done / 2026-10-22T15:42:45Z
- Plan-Write / done / 2026-10-22T16:02:31Z
- Plan-Bouncer / stuck / 2026-10-22T16:03:01Z
- Plan-Burler-Review / stuck / 2026-10-22T16:12:06Z
