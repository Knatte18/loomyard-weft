# Card 1 — lym-files-cache-rule

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-lym-files.md`
- `internal/lymdirs#DotLymDirName`

**Uses:**
- `internal/lymdirs/doc.go`

**ImpactSummary:** the `PATTERN-lym-files` rule text and the `DotLymDirName` doc comment say everything never-tracked under `.lym` is a cache; no code behavior changes.

**Intent:**
Make the declared `PATTERN-lym-files` change: everything never-tracked under `.lym` is a cache that a fresh machine rebuilds or starts without.

In `PATTERN.md`, replace the `PATTERN-lym-files` overview line with exactly this line:

```markdown
- `PATTERN-lym-files` — Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test) — [background](pattern/PATTERN-lym-files.md)
```

In `pattern/PATTERN-lym-files.md`, make the opening rule line carry the same rule text.
Reword the "Nothing a run, the orch or a landing depends on lives only under `.lym`" bullet to match it:
a run's decisions (its cancel and pause intents, its pull-request record, its fixer attempt counts) are tracked records under `_lym`.
Everything else run-related under `.lym` is a cache that reads as empty when absent and refuses nothing.

The doc comment of `DotLymDirName` says the same: the directory holds caches a fresh machine rebuilds or starts without, never a run's decision.

**Commit:** 1: PATTERN-lym-files: everything under .lym is a cache a fresh machine starts without
