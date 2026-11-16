# Card 13 — lym-files-cache

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-lym-files.md`
- `internal/lymdirs#DotLymDirName`

**Uses:**
- `internal/lymdirs/doc.go`

**Intent:**
Record that `.lym` holds only a cache a fresh machine rebuilds or starts without, now that the cancel and pause intents, the pull-request record and the fixer counts are tracked (D3).
`PATTERN.md`'s `PATTERN-lym-files` line becomes, whole:

```markdown
- `PATTERN-lym-files` — Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test) — [background](pattern/PATTERN-lym-files.md)
```

In `pattern/PATTERN-lym-files.md` the entry's summary line takes the same rule text, and the "Durable and ephemeral state" bullet that says nothing a run depends on lives only under `.lym` adds that every never-tracked file is a cache: an absent one reads as empty and refuses nothing, and a fresh machine starts without it or rebuilds it.
`lymdirs.DotLymDirName`'s doc comment says the same in one sentence.

**ImpactSummary:** documentation only: the PATTERN entry, its background and one doc comment.

**Commit:** 13: PATTERN-lym-files reads .lym as a cache a fresh machine starts without
