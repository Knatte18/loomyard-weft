---
round: 1
focus:
  - "internal/boardengine/handles.go `worktreeWindowTitle`: check each row of the Goal's title table against it (task in project, task with no project, task under a beam, beam itself, prime, orch, orch in a beam), for both the VS Code and the terminal form. Settle whether a beam's own entry, and a task whose beam is closed or missing, resolves to the right `<P#>`."
  - "Every place a title string was formed before the diff (grep for `ToUpper(shortname)`, `TitleWithHandle`, `--title`, `window.title`, `workspace`): settle whether any still forms a title without `consoletitle`, and whether `TitleWithHandle` is left with callers or dead."
  - "internal/reedcli/window.go and the reed templates (`template_posix.yaml`, `template_windows.yaml`): settle whether an explicit `--title` still wins, whether the no-worktree and unresolvable-prime fallbacks behave as the doc comments claim, and whether the template change leaves another consumer of `{slug}` or `--title` inconsistent."
  - "Goal limit 'no other part of lym reads a title back; a part that does is found first and named in the landing note': settle whether the diff or `_lym/landing/summary.md` shows that search, and whether any reader of the old `SHORTNAME: slug` or `T4: slug` form (batten, ideengine look, window matching, tests) is left broken."
  - "The new `internal/consoletitle` leaf and its `pattern/PATTERN-leaf-packages.md` change: settle whether the new import edges (`boardengine` and `fabricengine` into `consoletitle`, `boardengine` into `fabricengine`) break the repository's layering rules or leaf-package constraints."
  - "Scope against the Goal: the Goal says the `runs` strand and the operator shell get no title of their own, and that the title is not rewritten later. Settle whether the diff adds a title for either or rewrites one."
  - "Tests: settle whether the new and changed tests in `consoletitle_test.go`, `handles_test.go`, `window_test.go` and the `ideengine` look and integration tests cover the added behaviour without duplicating each other, and whether any asserts only static prose."
---

Round 1 focus, set from the Goal's title table, its Limits, and the diff's stat and core files.
