# The edit directive cat, head and sed -n clause has no effect

`edit-directive-cat-clause` · Note · enhancement, stencil

## Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 4.
Every sampled darn writer, code reviewer and plan writer read its first files with `cat`, batched with other lookups in one Bash call, against the edit directive's clause on `cat`, `head` and `sed -n` (`contracts/stencils/edit/edit-directive.md` and `contracts/stencils/engine/claude/edit-directive.md`).
None was refused and none caused a visible fault; the calls are the batching the same prompt asks for.

## Idea

The operator chooses: delete the clause, or add a shell guard row that refuses `cat <path>` with a pointer to Read, which turns one batched call into several parallel Reads at the same turn count.
