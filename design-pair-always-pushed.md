# Agents commit through lyx fabric, which pushes warp and weft together

`pair-always-pushed` · Note · bug, loom

A task branch should be on the remote at all times (operator, 2026-10-06), so that a crash, a lost machine or a look from another machine never meets work that exists only locally.

Seen 2026-10-06 in `test-suite-prune-rest`: the warp branch was 19 commits ahead of `origin/test-suite-prune-rest` while its weft was in sync.
The last warp push was at 15:40 by Webster (`internal/websterengine/sequence.go` pushes after its batches); the eleven Webster-Burler fix commits from 19:11 on and a merge of main at 19:52 stayed local until Publish.

Cause: agents commit warp with their own `git commit`, and pushing is left to whichever loop owner happens to push.
A Burler round commits each fix and never pushes (`internal/burlerengine/doc.go`, `FixScopeSource`, and its prompt in `internal/burlerengine/prompt.go`).
`lyx fabric commit`, `push` and `sync` cover the weft only, so a weft push never carries the warp along.

Operator direction: every agent in loomyard commits through a `lyx fabric` verb, never a bare `git commit`, and that verb commits and pushes the warp together with the weft, so the remote is never behind.
