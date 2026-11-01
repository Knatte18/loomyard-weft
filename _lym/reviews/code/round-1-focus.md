---
round: 1
focus:
  - "internal/boardengine/render.go recipeCell and entryCell: does every path that renders a Task row (Running, Ready, layers, Draft) reach the new Recipe cell with the profile, and does any other renderer or consumer of the old `profile: <name>` bullet remain (grep for `profile: `)?"
  - "internal/boardengine/render.go recipeCell: is the profile escaped through cellText like the recipe, and is the empty-recipe plus profile case (tapestry · profile) handled the way the board entry's Idea asks?"
  - "internal/boardengine/render_test.go: the golden test covers only a darn task with a profile; does any behavior the diff adds (empty recipe with a profile, recipe alone without a profile, pipe escaping in the profile) have no coverage, and is any added coverage a duplicate of an existing test?"
  - "internal/boardengine/doc.go: does the changed README-layout paragraph match what render.go now does, and do other docs, specs or stencils under the repo that describe the profile bullet need the same change in this diff?"
---

Small diff in one package. The measuring stick is the board entry's Idea under `## Goal`: recipe and profile together as `tapestry · light`, recipe alone when no profile, profile bullet under the title removed.
