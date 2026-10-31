# Comment cleanup: parallel agents fix stale comments across the repository

`comment-cleanup` · Note · enhancement, fabric

## Seen (operator and hub orch, 2026-10-31)

Comments in the Go source go stale as the code moves on.
Two found today: `internal/fabricengine/weftwiring.go` still speaks of a future `_lym/raddle/` squash-merge-back, though raddle will never be built; and many comments spell the `_board` path instead of naming its constant (note `bolt-dir-rename`).
There are surely more, in every package, against `scribe:prose`'s rule never to pin a perishable specific.

## Idea

A comment-cleanup campaign over the whole repository:
- An orchestrating agent splits the work by package and hands each package to a worker agent, many in parallel.
- A worker reads its package's code and comments and fixes each comment that no longer matches the code, names something gone or planned and dropped, or pins a perishable specific: a count, a list of current callers, a path or name spelled where a constant or identifier should be named.
- Comments only: a worker changes no code.
- Haiku 5.5 is a candidate worker model: fast and cheap while its context stays under about 100k, which a single package's read fits (operator, 2026-10-31).

Related: note `hardener` is another autonomous campaign run by an orchestrating agent, and note `loom-beam-darns` runs a large task's pieces as darn runs in a beam.

## Open

- How the campaign runs under lym: as darn runs per package inside a beam, or as one run whose agent spawns the workers.
- How a worker's result is checked before it lands, and by which model.
- Whether doc.go files, stencils and docs are in scope, or Go comments only.
