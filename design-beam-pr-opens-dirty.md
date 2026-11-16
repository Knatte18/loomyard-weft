# lym beam pr open opens a beam PR that conflicts with its parent

`beam-pr-opens-dirty` · Task · bug, batten, fabric

## Seen (adopt-build beam orch, 2026-11-16)

`lym beam pr open` opened beam `adopt-build`'s pull request #132 ready for review while it was "dirty", conflicting with `main`, and while a merge-in member still had to land.
The operator could have squash-merged a pull request whose work was still landing.

## Decision (operator, 2026-11-16)

A beam's pull request must not be opened before its conflicts with the parent are fixed.

## Idea

`lym beam pr open` refuses, or runs the merge-in first, while the beam conflicts with its parent or a member is still landing;
or it opens the pull request as a GitHub draft and marks it ready only at `lym beam ready` or once it is mergeable.
