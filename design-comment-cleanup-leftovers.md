# Defects the comment cleanup found but left: stale headings, dead helper, stencil paths, joint edits

`comment-cleanup-leftovers` · Task · bug, config, cwd, stencil, docs · done

## Seen

The `comment-cleanup` landing (`f7465bd8`, 2026-11-07) edited comments and prose only, and lists what its workers found but left:

- Stale headings in the `configengine` doc.
- `lymcwd.ResolveWithAnchor` has no non-test caller.
- Stencil paths under `{{.specs_dir}}/loom/` name no existing folder.
- Comment claims kept as written because a worker could not confirm them against the code.
- Boundary fragments where two workers' spans met.
- Files edited by more than one worker at once, among them tests in `cmd/lym`, `internal/adoptcli` and `internal/websterengine/render.go`; the commit asks a reviewer to read their diffs whole.

## Idea

One darn run settles the list: fix the stale headings and stencil paths, remove `ResolveWithAnchor` or give it its caller, confirm or correct each kept claim, and read the jointly edited files whole for broken or doubled comments.
The commit body of `f7465bd8` names where each was found.
