# finalize: the discrepancy-document conflict shape

`finalize-discrepancy-doc` · Note · enhancement, finalize, fabric

From `manifest/roadmap.md` Someday.
Some divergences cannot be expressed as a git conflict at all, so there are no markers to hand a resolving agent; the answer is a precomputed document describing the disagreement instead.
Only the ordinary-git-conflict shape shipped (`internal/mergeresolve`).
`PullResult.PatternResidue` was this shape for the history-rewrite case; `pattern-init` retired it on 2026-10-04, so design the document once for every such case, whenever this is picked up.