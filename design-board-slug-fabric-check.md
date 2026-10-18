# board accepts a task slug that batten Worktree-Create refuses (ends in -weft)

`board-slug-fabric-check` · Note · bug, board, fabric, batten

Imported from [issue #724](https://github.com/Knatte18/loomyard/issues/724).

Seen 2026-10-18: task orch-rules-in-weft was promoted and started; batten blocked at Worktree-Create with invalid slug, since a slug ending in -weft is reserved for weft worktrees. Workaround: cleared the run claim with lyx board set-status, renamed the entry to orch-rules-files with lyx board merge, started a new run; the old blocked batten record _lyx/shed/orch-rules-in-weft stays, since lyx batten cleanup skips blocked runs. Expected: lyx board upsert/promote refuses a task slug fabric cannot use, naming the rule, and a blocked run whose board entry is gone can be cleaned up.