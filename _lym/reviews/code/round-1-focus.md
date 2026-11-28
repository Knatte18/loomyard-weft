---
round: 1
focus:
  - "internal/runprofile/reinterpret.go, node rewrite: can any valid hub profiles.yaml using anchors, aliases or merge keys come out unparseable, lose operator values, or differ from what D7 asks? Check replacing a jacquard or default node that holds an anchor referenced elsewhere, anchored or aliased blocks and profiles shared between names, and merge keys inside a profile, a block or defaults."
  - "ReinterpretJacquardBlocks, degenerate shapes: what happens with a null, empty or non-map tapestry block, loom module or discussion area, an existing jacquard block where the profile holds no tapestry block, and a missing or null defaults entry? Does each outcome match D7, and does the rewritten file still pass the profiles load check?"
  - "The reconcile path (internal/hubreconcile/walk.go, applyReinterpretations) with the new entry: after the template follow, the arras rewrite and this one, is the result right on a first reconcile and on later ones? Check profiles.template-base interaction, a profile or default the operator removed, a file with no base, and what happens when the rewritten file fails its load check or a line is only flagged (does the one-time marker still get written)."
  - "Reported lines: does each rewrote and flagged line state what happened truthfully, including the dropped-default and merge-key cases and a profile whose jacquard block is already present, and is each named way forward one that works?"
  - "internal/runprofile/reinterpret_test.go and the other changed tests: is every behaviour of the D7 acceptance criterion covered, and is any new test or assertion redundant with another test, tautological, or an assertion on static prose?"
  - "Prose against code: do the doc comments, doc.go, the template.yaml header, any repository document naming the profiles' recipe blocks, and _lym/landing/summary.md (its Merge danger door and blast radius included) match the diff and the repository's own rules?"
---

Seed pass: no round has been judged yet.
The entries come from the decision record's D7 and the shape of the diff, and state no verdict.
