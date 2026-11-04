# Dobby and the orch watcher leave their child processes as zombies

`dobby-zombie-children` · Task · bug, dobby, orch, tender, process

## Seen (hub orch, 2026-11-03 ~20:50)

`lym tender status` listed about 170 `batten` processes as `owned`, `orphan: false`, under slugs that landed hours or a day ago.
`ps` shows every one as `[lym] <defunct>`: zombies whose parent never reaped them.

Zombies per parent at that time:

- 170 under the loomyard prime's `lym beam dobby` (up 25 h).
- 31 under the loomyard `lym orch watch` (up 25 h).
- 14 under the `cs` hub's `lym beam dobby` and 15 under its `lym orch watch` (up 5.5 h).

So both dobby and the orch watcher start child `lym` processes and never `Wait` on them, and the tender registry reads a zombie as a live owned process.

## Idea

- Dobby and `lym orch watch` reap every child they start (wait on it, or detach it so init reaps it).
- The process registry treats a zombie (state `Z`) as gone, so `lym tender status` and `lym tender reap` stop listing landed runs as live.
