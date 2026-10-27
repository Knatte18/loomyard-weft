# lym ide seed with no slug seeds the worktree it runs in

`ide-seed-current-worktree` · Note · enhancement, ide

## Seen

`lym ide seed <slug>` requires a slug, and its help names only task worktrees.
A fresh `lym fabric clone` leaves the prime with no `.vscode/`, while every task worktree gets one from its run; seeding the prime took `lym ide seed loomyard` with the prime's name spelled out.

## Idea

The slug becomes optional: with none, `lym ide seed` seeds the worktree the current directory sits in, the prime included, with the prime's green title bar (issue #882).
The help text names the prime as well as task worktrees.
`fabric clone` stays IDE-free.
