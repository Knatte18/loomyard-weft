# Comment cleanup: parallel agents fix stale comments across the repository

`comment-cleanup` · Task · enhancement, fabric · done

## Seen (operator and hub orch, 2026-10-31)

Comments in the Go source go stale as the code moves on.
Two found today: `internal/fabricengine/weftwiring.go` still speaks of a future `_lym/raddle/` squash-merge-back, though raddle will never be built; and many comments spelled the `_board` path instead of naming its constant (the rename to `_bolt` in `bolt-dir-rename` has since replaced the spelling itself).
There are surely more, in every package, against `scribe:prose`'s rule never to pin a perishable specific.

## Goal

A comment-cleanup campaign over the whole repository.

- A worker fixes each comment or text that no longer matches the code, names something gone or planned and dropped, or pins a perishable specific: a count, a list of current callers, a path or name spelled where a constant or identifier should be named.
- Workers change comments and text only, never code.

## Design (operator, 2026-10-31)

- One darn run, whose agent orchestrates the whole campaign: it splits the work and spawns the workers, many in parallel; not one darn per package, which would be far too many runs.
- Scope: Go comments and docstrings, each package's `doc.go`, the stencils and the docs.
- Models by scope: Haiku 5.5 takes the comments and docstrings of code files; `doc.go`, stencils and docs need a stronger model, Sonnet 5.5, as the `docs` profile's writer is.
- Every Haiku 5.5 worker is scoped to modules so that its context stays under 100k tokens, since below that Haiku 5.5 is cheap (operator, 2026-11-04): one package per worker, a large package split by file groups, small packages grouped while the sum stays under the bound.
  The writer sizes each worker's scope before it spawns it.
- No review step of its own: most comments are assumed sound, and this run is itself the check on them; the darn's usual Code-Review is enough.
- The run may start beside `tier1-test-revision`, which only writes a report (operator, 2026-11-06).

## Workers (hub orch, 2026-11-03)

The writer spawns its workers as subagents through its own Agent tool, with the tool's model override set to `haiku` or `sonnet` by the scope above; no lym verb is needed.
Each worker gets its scope from the bound above, edits only comments and text in it, and reports what it changed; the writer commits.

Related: note `hardener` is another autonomous campaign run by an orchestrating agent.
