---
round: 1
focus:
  - "Scope against the Goal: the symptom is seen in `lym board list --text`, but the diff changes only `viewProjects` in internal/boardengine/view.go. Settle whether the list --text path shares this code or builds its own project grouping that still files a done beam task under P0."
  - "internal/boardengine/view.go `viewProjects`: a done project's `Parent` comes from `b.projectHandle(twl.Task)`. Settle whether that returns the project whose Done section lists it, including a done project nested under another done project, and not P0 by fallback."
  - "internal/boardengine/view.go `viewProjects`: the new loop appends every done project after the open walk. Settle whether a done project can also be reached by `walk` through `childProjects`, which would list it twice, and whether the consumers of `Projects` (board.html, the viewer page) order or index by handle in a way the appended done entries break."
  - "internal/boardengine/view.go: the closure `add` takes `Task`, and the old walk passed `child.Task` to `designFile`. Settle whether `DesignFile`, `Brief`, `Labels` and `Slug` come out the same for open projects as before the refactor."
  - "internal/boardengine/view_test.go: the fixture gains T11, a done task in open project gamma, and a done project P3 is expected in `Projects`. Settle whether any other test over `viewFixture` (rendered HTML, manifest, viewer page, JSON data shape) is affected by the extra entry, and whether the new subtest asserts the done-project ordering and parent independently of the code's own computation."
---

Seed focus only; the questions name where to look and leave the verdicts to the reviewer.
