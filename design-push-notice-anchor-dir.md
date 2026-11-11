# FABRIC push-failure notice names the worktree root, which fabric push refuses in an anchored hub

`push-notice-anchor-dir` · Task · bug, fabric

Depends on: [`adopt-here-archives`](design-adopt-here-archives.md)

Imported from [issue #98](https://github.com/Knatte18/loomyard/issues/98).

In a hub whose anchor is a subdirectory (Models, anchor src/csharp/NORCE.Models), the mo:FABRIC notice for a failed async push says: way forward: run `lym fabric push` in <hub>/knut-rheology-merge-in. Running it there fails with: cwd is outside the recorded fabric anchor subtree: cwd <hub>/knut-rheology-merge-in does not equal the anchored directory <hub>/knut-rheology-merge-in/src/csharp/NORCE.Models (recorded by .lym-anchor).

Workaround: run `lym fabric push` in the anchored directory instead; it pushed.

Expected: the way forward names the anchored directory (or uses env -C with it), so the command it gives works as written.