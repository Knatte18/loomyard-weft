# Retro prompt fixes: pin tool facts in decisions, drop the cat clause

`retro-stencil-fixes` · Task · enhancement, stencil, loom · running · Drift-Baseline

Two prompt-text changes from the retro of 2026-11-17 (`docs/research/retro/retro-20261116-115411/report.md`), both in `contracts/stencils/`.

# Rubric: pin the tool facts a decision rests on

### Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 2.
In `decision-record-replace`, three sessions in different rounds (reviewers 91782c57 turn 20 and e44fb294 turn 14, fixer 2c77f780 turn 21) each built a scratch git repository to learn what `git commit --allow-empty -- <path>` commits when another path is staged.
The record stated the behaviour its decision relied on without the observed result, so each round repeated the experiment.

### Idea

A rubric line in `contracts/stencils/loom/loom-rubric-discussion-review.md`: a decision that rests on how git or another tool behaves states the observed result and where it is pinned.

# Delete the cat, head and sed -n clause

### Seen (retro, 2026-11-17)

From `docs/research/retro/retro-20261116-115411/report.md`, proposal 4.
Every sampled darn writer, code reviewer and plan writer read its first files with `cat`, batched with other lookups in one Bash call, against the edit directive's clause on `cat`, `head` and `sed -n` (`contracts/stencils/edit/edit-directive.md` and `contracts/stencils/engine/claude/edit-directive.md`).
None was refused and none caused a visible fault; the calls are the batching the same prompt asks for.

### Idea

The operator chooses: delete the clause, or add a shell guard row that refuses `cat <path>` with a pointer to Read, which turns one batched call into several parallel Reads at the same turn count.

Decision (operator, 2026-11-18): delete the clause; no guard row.
