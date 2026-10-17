# lyx board rename: change an entry slug in one write

`board-rename` · Note · enhancement, board

## Seen

`lyx board` has no verb that changes an entry's slug.
Renaming `enclose-kickstart` to `enclose-checks` (2026-10-17) took `get --body` to a file, a `merge` with `remove_slugs` and an upsert carrying the old entry's title, brief, kind, labels and `created`, then a manual fix of the old slug in another entry's body.

## Idea

A verb `lyx board rename {slug, to}` that renames an entry in one atomic write, keeping its body, kind, labels, priority, issues, `created` and every other field, and rewrites the old slug in every other task's `depends_on`.
It refuses a run-held entry, as the run lock does for any scope change, and a target slug that exists or breaks the slug length limit.
It reports the entries whose body or brief still mention the old slug, since prose references are not rewritten.
