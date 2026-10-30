---
round: 2
focus:
  - "internal/boardengine/render.go, `projectSections` and `childProjects`: settle whether every open project gets a README section when a done project sits anywhere in its parent chain, and whether the depth-first section order still holds."
  - "internal/boardengine/doc.go README paragraph and the `projectSections` comment: settle whether the stated order of root-level project sections matches what `renderReadme` and `childProjects` actually apply."
  - "internal/boardengine/render_test.go golden rows for project sections: settle whether a member's priority cell and linked slug are pinned, and whether a row covers an open project under a done project."
---

Entries name where to look and the question to settle; the reviewer judges the committed diff afresh.
