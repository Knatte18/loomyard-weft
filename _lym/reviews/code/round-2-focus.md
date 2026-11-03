---
round: 2
focus:
  - "internal/boardengine/dependency.go `met` and `standsFor`, and internal/boardengine/layer.go `getDepth` and `detectCycleDFS`: settle whether each terminates on a stored project membership loop, including a project naming itself, and whether the tests build such a loop."
  - "internal/boardengine/render.go `afterCell`: settle whether the README render tests pin a met project dependency (cell empty) and an unmet project dependency (cell holds the handle), so a fallback to `isDone` would fail a test."
  - "internal/boardengine/siblings.go `addedSiblings` met filter: settle whether a siblings test row reaches the branch that drops an added dependency on a plain project whose landing members are all done."
  - "internal/boardengine/render.go file header: settle whether its description of the After column matches the `afterCell` doc comment and doc.go."
---

The focus lists where the round-1 gaps were; it states no expected outcome.
