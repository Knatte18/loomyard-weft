# Comment cleanup: parallel agents fix stale comments across the repository

`comment-cleanup` · Note · enhancement, fabric

## Seen (operator and hub orch, 2026-10-31)

Comments in the Go source go stale as the code moves on.
Two found today: `internal/fabricengine/weftwiring.go` still speaks of a future `_lym/raddle/` squash-merge-back, though raddle will never be built; and many comments spell the `_board` path instead of naming its constant (note `bolt-dir-rename`).
There are surely more, in every package, against `scribe:prose`'s rule never to pin a perishable specific.

## Idea

A comment-cleanup campaign over the whole repository.
- A worker fixes each comment or text that no longer matches the code, names something gone or planned and dropped, or pins a perishable specific: a count, a list of current callers, a path or name spelled where a constant or identifier should be named.
- Workers change comments and text only, never code.

## Design (operator, 2026-10-31)

- One darn run, whose agent orchestrates the whole campaign: it splits the work and spawns the workers, many in parallel; not one darn per package, which would be far too many runs.
- Scope: Go comments and docstrings, each package's `doc.go`, the stencils and the docs.
- Models by scope: Haiku 5.5 takes the comments and docstrings of code files, being fast and cheap while its context stays under about 100k, which one package's read fits; `doc.go`, stencils and docs need a stronger model, Sonnet 5.5, as the `docs` profile's writer is.
- No review step of its own: most comments are assumed sound, and this run is itself the check on them; the darn's usual Code-Review is enough.

Related: note `hardener` is another autonomous campaign run by an orchestrating agent.

## Open

- How the orchestrating agent spawns its workers under lym, with Haiku 5.5 as their model.
