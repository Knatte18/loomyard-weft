---
round: 1
ledger:
  - key: render-output-path-guard-and-mkdir-dropped
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: concurrency-test-comment-claims-atomic-home
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: render-doc-generalises-watcher-behaviour
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 found that `RenderToDisk` lost `fsx.PathGuard` and parent-directory creation when it moved to an in-place `os.WriteFile`.
The fix calls `PathGuard` and `os.MkdirAll` before the write and adds test rows for a nested and an escaping readme.
Two consistency findings follow: a stale doc comment in the concurrency test, and an over-general watcher claim in the `RenderToDisk` doc comment.
