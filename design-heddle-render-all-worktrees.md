# heddle set/remove should re-render file targets in every live worktree of the hub

`heddle-render-all-worktrees` · Note · enhancement, heddle

Imported from [issue #103](https://github.com/Knatte18/loomyard/issues/103).

lym heddle set and lym heddle remove save the entry and render HEDDLE.md, but leave every existing worktree's rendered files (CLAUDE.local.md, AGENTS.override.md, .claude/settings.local.json) stale until the worktree is made, taken in, or rendered by hand with lym heddle render.

Seen: a new repository entry with target claude-file was set from a beam worktree. CLAUDE.local.md in that worktree, in the prime worktree and in a running task worktree all lacked the entry until lym heddle render was run in each one by hand.

An instruction entry usually exists because agents must follow it now (for example, a module that is out of scope), so agents in running task worktrees keep working without it, and nothing tells the caller which worktrees are stale.

Proposal: after a successful set or remove, render every file target into every live code worktree of the hub (prime, beams, task pairs), and report the rendered and failed worktrees in the verb's output. Running Claude sessions then take the change at their next compaction, as today; an optional flag to skip the fan-out could cover bulk edits.