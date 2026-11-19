---
round: 2
focus:
  - "Card 2 (`_lym/plan/02-entry-revision-check.md`) reads an empty expected revision as a create, and card 4's `applyWrite` (`_lym/plan/04-board-server-routes.md`) forwards each op's request revision. Settle whether every op but `create` refuses an empty `revision` before any write, with a named input and a named `TestServer` row that reaches the refusal, and whether card 4's Bound covers the path."
  - "Card 5's `Run` exit order (`_lym/plan/05-board-server-lifecycle.md`) must stop taking requests, release the lock, wake the waiters, then wait for requests in flight. Settle whether established keep-alive connections stop taking requests before the lock is released, and whether `http.Server.Shutdown`'s error for an already-closed listener is handled."
  - "Cards 7 and 9 (`_lym/plan/07-vscode-tasks-and-keys.md`, `_lym/plan/09-beam-start-workspace-task.md`) add routing edges: user-global keys running a task by label, and a process-wide hook adding a `process` task to regenerated workspace files. Settle whether each Intent states what the change lets through and what bounds it."
  - "Any text added to card Intents for the board's package-tree docs, the render and replay comments, and `internal/runproc/doc.go` must pass the plan gate: a file target beside a member target of that file is refused by `redundant-file-target`, and an `Extends` on a ref whose existing sentence is rewritten belongs under `Edit`."
---

Focus targets the design-class areas of round 1 and the surfaces the fixes touch.
