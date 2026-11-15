# Pairs a bench or a cancelled run left behind are never torn down and stay in the workspace

`stale-bench-pairs-linger` · Note · bug, batten, fabric, cleanup, workspaces

## Seen (operator, 2026-11-15)

Two throwaway bench pairs, `pwc-opus` and `pwc-fable`, still stand three days after their bench landed (report `384d7cb7`, 2026-11-12).
Their worktrees and weft checkouts sit beside the prime, and the root `loomyard.code-workspace` still lists them as folders, so they show up in the operator's VS Code.
The kept pair of the cancelled raddle run `claude-account-weights` stands the same way, and the `rerun` pair from the advisor trial fails to respawn on every `lym tender resume`.
Nothing tears down a pair that a bench or a cancelled run left behind, and nothing tells the operator it stands.

## Idea

A pair made for a bench or trial belongs to the task that made it, and that task's landing tears it down with its own pair.
A cancelled run's kept pair is torn down after a set time unless the operator keeps it by name.
The workspace file drops a folder once its worktree is gone.
