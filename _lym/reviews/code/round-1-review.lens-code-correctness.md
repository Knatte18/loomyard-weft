```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I walked each changed function and branch with a concrete input.

**Cases run**

- `dependsOnList` (`internal/boardcli/cli.go`), correct on every input I ran:
  - `nil` (JSON null) gives `[]string{}`.
  - `[]` gives an empty non-nil slice.
  - `"q"` gives "depends_on must be an array".
  - `[1]` gives "depends_on elements must be strings".
  - `["a"]` gives `["a"]`.
- `set-deps` after its refactor, correct. A null value still clears the list and a missing key is still refused by `setDepsPayload`'s required key. The new `dependsOn, err :=` compiles because `dependsOn` is new, and the later `siblings, err :=` also introduces a new variable.
- `promote` CLI, correct:
  - An absent `depends_on` key leaves `dependsOn` as `nil`, so behavior is unchanged.
  - `"depends_on": null` or `[]` gives non-nil empty, which clears a note's deps and matches an already-task entry that has none.
  - A bare handle such as `S3` yields `m` holding only the handle and passes `promotePayload`.
  - `slug` plus `id` is still refused by the exclusive group.
  - A dependency error is returned before any write.
- `Store.promoteTo`, task branch, correct:
  - A `nil` `dependsOn` returns the task unchanged.
  - An equal list returns it unchanged. `slices.Equal` treats a `nil` list and an empty list as equal, so the two never get refused as a difference.
  - A differing list gives the `set-deps` refusal and writes nothing.
  - An equal set in a different order is refused. That is strict but harmless.
- `Store.promoteTo`, note branch, correct:
  - `dependsOn` replaces `incoming.DependsOn` before `validateWrite`, so these are all refused with no state changed: a dangling slug, an isolated dependency, a note as a dependency, a self-dependency, and a cycle.
  - In the `recipe == ""` raddle-on-sketch path, the deps go into the same `s.tasks[i] = incoming` assignment that releases the task, so the dobby never sees the task released without them.
  - Slice aliasing of the caller's `dependsOn` is harmless.
- `Board.PromoteTo` and `Board.Promote`, correct:
  - The `writeReportingSiblings` swap keeps the same `boardCriticalSection` call with a nil second argument.
  - On error it returns `Task{}, nil, err`.
  - The `noWrite` path leaves the store unchanged, so `addedSiblings` returns no siblings.
  - `Promote` discards the siblings and passes a `nil` `dependsOn`, so its behavior is unchanged.
- `siblingNotice` on `promote`: an empty list gives `""`, so no `notice` key is added. Correct.
- `Store.Promote` passes `nil, nil`. Correct.

**Findings:** none.

**Tool calls:** none. The inherited context already held every definition I needed.
