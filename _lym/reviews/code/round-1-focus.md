---
round: 1
focus:
  - "internal/boardengine/dependency.go and store.go: is a dependency met exactly when its entry is done for every kind (beam landed into its parent, plain project with every landing member landed or marked done, task landed), and does a missing or never-landing slug stay unmet?"
  - "internal/boardengine/dependency.go: does the project-as-union-of-members expansion terminate on nested projects and membership loops, and does the cycle check in store.go still leave a membership-only cycle to the project rules' refusal as the second commit claims?"
  - "internal/boardengine/layer.go, render.go and internal/battencli/dobbyreads.go: do Dobby's hold, the layers, and the README After and Ready output treat a project dependency like a task one, with no path still reading the raw depends_on slugs or still refusing a project?"
  - "internal/boardengine/siblings.go: does the set-deps overlap notice treat a project as the union of its members, and does the notice cover project dependencies without false positives for a project and its own member?"
  - "internal/boardengine/project.go: the removed code of 24 changed lines, is the refusal 'nothing depends on a project' fully gone, and is every other project rule (membership, beam, done marking) unchanged in behavior?"
  - "Tests in dependency_test.go, layer_test.go, project_test.go, siblings_test.go and dobbyreading_test.go: does each new behavior have coverage, and does any new test duplicate an existing test's coverage?"
  - "internal/boardengine/doc.go and any spec or doc naming the old project-dependency refusal: does the diff update the docs the repository's rules name for this kind of change, and do new doc comments follow the repository's comment conventions?"
---

Seed focus from the decision record's `## Goal` (the only measuring stick: the record's other sections are empty) and the diff's file list.
