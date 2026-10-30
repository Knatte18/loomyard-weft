```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1, F2 and F3 proposals as the fixer carried them out (commit e9032bde).

**LOW — `internal/boardengine/render.go` `childProjects` and `doc.go:171`: open tasks and notes under a done project are still in no README section.**
- The fix rescues only open projects under a done project. Open tasks and notes whose `Project` is a done project's slug are listed by none of these:
  - `projectSections` emits sections for open children only.
  - Prime's `entrySections(Task{})` keeps only entries with `twl.Project == ""`.
  - A done project's own file lists them, but that file is not the README.
- It is the same situation F1 describes: prune keeps a done project while an entry still names it.
- `doc.go:171` still says "so every open entry shows in the README". That is false for this case, and the fix left the sentence as it was.
- A real fix would either render a section for such entries or drop the claim. This is outside the review's stated premise, which names open projects only, so it is LOW.

**LOW — `render_test.go`, the "each project renders to its own file" golden row: the rescue case checks the README only.**
- `project-old.md` is not in the `files` map, so the `Projects` section that a done project's file now holds for its open child `orphan` is not pinned.
- `project-orphan.md` is added to `projectFiles`, but its "Under:" line is not checked either.

No other case I built against the proposals breaks them:
- **F1:** an open project under a done project under another done project resolves to the root, because the immediate parent is done. An open project under an open project under a done one is reached by recursion from its open parent, so no section appears twice. The done parent is never recursed into, because `childProjects` skips done projects.
- **F2:** the reworded text matches the code. The root passes the zero Task, which has no order, so its children run by handle.
- **F3:** the golden passed unchanged in order with `H`/`L` on the Ship members and the `design-second.md` link on `second`.

No further tool calls; everything I needed was already in context.
