# A task branch is pushed after every step that commits

`pair-always-pushed` · Note · bug, loom

A task branch should be on the remote at all times (operator, 2026-10-06), so that a crash, a lost machine or a look from another machine never meets work that exists only locally.

Seen 2026-10-06 in `test-suite-prune-rest`: the branch was 19 commits ahead of `origin/test-suite-prune-rest`.
The last push was at 15:40 by Webster (`internal/websterengine/sequence.go` pushes after its batches); the eleven Webster-Burler fix commits from 19:11 on and a merge of main at 19:52 were never pushed.
By design a Burler round commits each fix and never pushes (`internal/burlerengine/doc.go`, `FixScopeSource`), and nothing after it pushes until Publish, so every review round's work stays local until the PR is opened.

Wanted: the loop owner pushes the task branch after every step that committed, not only Webster, so the remote is never more than one step behind.
