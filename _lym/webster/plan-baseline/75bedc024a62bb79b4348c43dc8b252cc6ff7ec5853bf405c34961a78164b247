# Card 17 — lym-files-pattern

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-lym-files.md`
- `internal/lymdirs#DotLymDirName`

**Uses:**
- `internal/lymdirs/doc.go`

**Intent:** Write the declared `PATTERN-lym-files` change: never-tracked state under `.lym` is a cache a fresh machine rebuilds or starts without.
`PATTERN.md`'s entry line becomes:

```
- `PATTERN-lym-files` — Placing a file, temp dir, lock or link: tracked under `_lym`, never-tracked under `.lym` as a cache a fresh machine rebuilds or starts without; named only by `lymdirs`; temp under `lymtemp`, lock paths via `lock`, links via `fslink`. (test) — [background](pattern/PATTERN-lym-files.md)
```

`pattern/PATTERN-lym-files.md`'s summary line takes the same rule text, and its Durable and ephemeral state section names what moved into tracked records:
the cancel and pause intents in the batten record, the pull-request record and the fixer attempt counts.
It says everything else run-related under `.lym` is such a cache: the merge record, the driver park marker and resume report, batten's per-row marks, the dobby's `beam/state.json`,
shuttle run directories, reed state, loom scratch, gate and verify records, the parent lock, the run process registry, the creel journal, adopt check views, failed-push records, the hibernate request and the pause record;
an absent cache reads as empty and refuses nothing, and a lost sent-notice memory can repeat a notice or a revive once.
It names the tracked values that hold only on the machine that wrote them, the Claude session ids and absolute paths, as read such.
`DotLymDirName`'s doc says the same of what lives under it.

**ImpactSummary:** One PATTERN line, its background file and one constant's doc comment change; no code behaviour.
