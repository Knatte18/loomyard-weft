# Fabric single-repo mode: the weft lives inside the warp repo, no piggyback repo

`fabric-single-repo-mode` · Task · sketch, fabric · starting · Open-IDE

Decision record: [Sketch](decision-fabric-single-repo-mode.md)

## What was seen

Every hub today needs two repositories: the warp (the code) and the weft (the piggyback repo that holds lym's housekeeping),
with `_lym` and `.lym` in each worktree as junctions into the weft.
For smaller work the operator wants lym's orchestration (creel, board, reed, runs) without setting up a second repository.
The warp/weft split is meant to be invisible from outside fabric, which this mode can use.

## Idea (operator, 2026-11-18)

A single-repo ("monorepo") mode: `_lym` and `.lym` are real directories in the warp repository, not junctions.
Everything outside fabric looks and behaves the same; only fabric knows where the weft lives.

## Open questions for its Discussion

- Where housekeeping commits go. Every weft commit already goes through `lym fabric commit` (or `sync`/`push`) as its own "weft sync" commit, and `.lym` is never committed and stays in `.git/info/exclude` as today; `_lym` must then not be in the exclude (operator, 2026-11-18); the question is only whether `_lym` commits sit on the code branch or on a separate branch of the same repo.
- What a landing does with `_lym` and `.lym`: excluded from the PR and the merge to the parent, or carried.
- Which parts of today's weft (records branch, board, per-task branches, push transport) need a single-repo counterpart.
- Whether code outside fabric assumes the split or commits `_lym` itself: several packages call git directly (`hubforge`, `websterengine`, `burlerengine` among them), most likely for warp commits; verify.

## Relation

The operator finds `fabricengine` disproportionately large for its job (36,000 production lines, about 1,100 functions;
`docs/research/rewrite/shrink-estimate-2026-11-18/report.md`) and wants this mode taken in if fabricengine is rewritten (note `clean-room-module-rewrite`).
