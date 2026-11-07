# git worktree list is 15 percent of tier 1 git processes and only reads .git/worktrees

`worktree-list-through-dotgit` · Note · enhancement, fabric

## Seen

`git worktree list --porcelain` is about 15 percent of all git processes in tier 1: 14,744 in an `internal/fabricengine` run and 22,636 in a whole pass (`docs/research/speed/tier1-test-revision-2026-11-06/report.md`).
It only reads `.git/worktrees/`, and `lym` calls it several times in each `Add` and `Remove`.
Its call sites are `internal/fabricengine/worktreelist.go` and `internal/websterengine/gitwrap.go`.

## Idea

Read `.git/worktrees/*/{gitdir,HEAD,locked}` in `internal/dotgit` behind `fabricengine.List` and the `websterengine` call.
A test checks the reader against `git worktree list --porcelain` on a repository with a detached, a locked and a prunable worktree, since that parity is the new behaviour to pin.

Expected: about 5 s off a tier 1 pass, and fewer processes in every `lym` verb that lists worktrees.

Wait for `comment-cleanup` to land before starting this (operator, 2026-11-07).
