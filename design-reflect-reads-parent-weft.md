# Friction-Reflect reads the parent weft when a halt names a dirty worktree

`reflect-reads-parent-weft` · Note · enhancement, loom, fabric

Seen (publish-recovery, 2026-10-10): Finalize blocked on "worktree dirty" from the prime weft's batten records (#501), and Friction-Reflect dismissed it as a process signal because the task worktree was clean.

Idea: Friction-Reflect also reads the parent weft's status (`lyx fabric status` in the parent) when a halt names a dirty worktree, so it attributes the halt to the right tree.
