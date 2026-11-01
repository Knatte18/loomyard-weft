---
round: 1
focus:
  - "internal/boardengine/render.go `projectSection`, `projectBody`, `entrySections`: trace heading levels for the README (Prime at 2) and a project file (header as level 1) through two or more nesting depths, and settle whether every Tasks, Notes, Done and subsection heading sits one level below its project's heading and caps at six without a malformed heading."
  - "`projectSection` member count: settle whether the count in a project's heading matches the 'open members' the doc comments and doc.go state, given that the members include open sub-projects and exclude done entries, and whether it agrees with what the project's own sections list."
  - "Behavior the design asks for against the diff: a running member of a project shows under that project's Running, the root's own entries come last in README.md, a project file shows only that project and the projects under it, and an open project under a done project still counts as under the root."
  - "Removed code: settle whether any helper, constant or column left unused by deleting `projectsSection`, `projectSections` and `projectsHeading` (for example `columnEntry`, `entryCell`, `emptySectionLine`, the `priority` option on `readmeTable`) is now dead, and whether any remaining doc comment or doc.go sentence still describes the removed flat member table, Projects bullet list or Prio column on project rows."
  - "internal/boardengine/render_test.go: settle whether each new or changed test covers behavior not already covered by another test in the file, whether the depth-six cap and the nested-project paths each have coverage, and whether any test asserts static prose."
  - "docs/overview.md and doc.go claims against the diff: settle whether the README and project-file layout they describe matches what `renderReadme` and `renderProjectFile` produce, including the ordering of sub-projects and the Prime-last placement."
---

Seed focus for round 1 of the review of `4ce42a8d..HEAD`, a single commit that replaces three render paths with one recursive project section.
