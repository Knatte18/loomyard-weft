```yaml
lens: fix-affected-set
outcome: reported
```

Findings: one LOW.

- **LOW, `internal/boardcli/doc.go:66`**: the `profiles` doc says "and `[]` when there is none". The envelope now carries `defaults` too, so the empty case is `"profiles":[]`, not a bare `[]`. The clause is stale beside the shape the fixer just wrote. Evidence: `cli.go:390` returns `{"profiles": entries, "defaults": loaded.Defaults}`, and the doc line starts with the full envelope.

F1 and F2: the search found no other site with the old output shape or the old `profile` field description.
- The other two mentions, `plugins/ly/skills/board/SKILL.md:47` and `docs/overview.md:358`, already describe `defaults` and per-recipe settings.
- The `upsert` help in `cli.go:487` is the only `profile` field description.

Unchecked: F3, F4 and F5 have no set premise. F3 and F4 are single coverage gaps, and F5 is a single comment.

Tool calls:
1. `grep` for the profiles output shape and the `profile` field description across `.go` and `.md` files: my context did not show sites outside the files the review named.
