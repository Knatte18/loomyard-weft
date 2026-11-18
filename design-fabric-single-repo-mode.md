# Fabric single-repo mode: the weft lives inside the warp repo, no piggyback repo

`fabric-single-repo-mode` · Note · feature, fabric

## What was seen

Every hub today needs two repositories: the warp (the code) and the weft (the piggyback repo that holds lym's housekeeping),
with `_lym` and `.lym` in each worktree as junctions into the weft.
For smaller work the operator wants lym's orchestration (creel, board, reed, runs) without setting up a second repository.
The warp/weft split is meant to be invisible from outside fabric, which this mode can use.

## Idea (operator, 2026-11-18)

A single-repo ("monorepo") mode: `_lym` and `.lym` are real directories in the warp repository, not junctions.
Everything outside fabric looks and behaves the same; only fabric knows where the weft lives.

## Open questions for its Discussion

- How housekeeping commits stay apart from code commits: separate commits on the same branch, a separate orphan branch in the same repo, or a ref namespace.
- What a landing does with `_lym` and `.lym`: excluded from the PR and the merge to the parent, or carried.
- Which parts of today's weft (records branch, board, per-task branches, push transport) need a single-repo counterpart.
- Whether code outside fabric assumes the split anywhere, which the invisibility promise rules out.

## Relation

The operator finds `fabricengine` disproportionately large for its job (36,000 production lines, about 1,100 functions;
`docs/research/rewrite/shrink-estimate-2026-11-18/report.md`) and wants this mode taken in if fabricengine is rewritten (note `clean-room-module-rewrite`).
