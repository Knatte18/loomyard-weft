# Card 1 — lym-files-cache-pattern

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-lym-files.md`
- `internal/lymdirs#DotLymDirName`
- `internal/lymdirs/doc.go`

**Intent:** Declare D3's split: everything run-related under `.lym` is a cache a fresh machine rebuilds or starts without, and every run decision lives in tracked records under `_lym`.

Replace the `PATTERN-lym-files` overview line in `PATTERN.md` with exactly this line:

```markdown
- `PATTERN-lym-files` — Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test) — [background](pattern/PATTERN-lym-files.md)
```

In `pattern/PATTERN-lym-files.md`, change the header sentence and the `## Durable and ephemeral state` rule to the same wording.
Reword the bullet "Nothing a run, the orch or a landing depends on lives only under `.lym`" so it says:
a run's decisions, its cancel and pause intents, its pull-request record and its fixer attempt counts are tracked records;
an absent `.lym` reads as empty and refuses nothing;
and a sent-notice memory lost with it can repeat a notice once on the new machine.
Keep the rest of the background as it stands.

Reword the doc comment of `DotLymDirName` and the matching sentence of `internal/lymdirs/doc.go`:
the directory holds machine-bound caches and locks that a fresh machine rebuilds or starts without.

**ImpactSummary:** Prose and two doc comments; no code behavior changes.

**Commit:** 1: lym-files-cache-pattern
