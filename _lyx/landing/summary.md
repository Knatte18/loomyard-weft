# boardengine: one board.json store with tiers, types and a roadmap README

The board becomes the roadmap.
`tasks.json` and `notes.json` give way to one store, `board.json`, shaped `{"version": 1, "entries": [...]}`.
Every entry carries a numeric `tier` and a `type`, and the README renders like `manifest/roadmap.md`: one section per tier, one `1.` item per entry.
Agents get `lyx board find`, a compact `--text` listing and payload shapes in every verb's `--help`, so they stop falling back to jq on the JSON files.

## What changes

- **Entry schema.**
  `tier` is 1, 2 or 3 (claimable, planned next, loose idea) and defaults to 3 on a new entry.
  `type` is one of `feature`, `bug`, `chore`, `design` and defaults to `feature`.
  The old recipe-carrying `type` field is now `recipe`, and `internal/battencli` reads the child recipe from it.
  `deferred` is retired; an upsert carrying it is refused with a message naming `tier: 3`.
- **Dependency rule.**
  An entry's dependencies must have a tier less than or equal to its own, checked on every write over the projected store, including tier changes.
  A dependency that is `done` is exempt.
  This replaces the old deferred rules; the dangling, isolated and cycle checks stay.
- **New verbs.**
  `promote` (default: one tier lower, skipping allowed, never equal or higher), `prune` (removes every `done` entry, strips it from `depends_on`, and the render removes its design doc), `find` (case-insensitive over slug, title, brief and body, done included), and `retire-legacy`.
  `list` and `find` take `--text` for one aligned line per entry in README order.
- **Aliases.**
  `lyx board notes <verb>` and `promote-note` stay as hidden aliases over the same store, so `notes list` lists every entry.
  `promote-note` on a tier-1 entry succeeds without a change.
- **README and design docs.**
  README sections are Planned, Next Up, Someday and Done (omitted when empty), each with a meaning line; the item shape follows `roadmap.md`, with no ids or layer letters.
  Design docs gain a header with a metadata line and a `Depends on:` line; the body is written unchanged.
- **Help and skill.**
  Every payload-taking verb documents its keys, types, required flags and an example.
  The new `ly:board` skill (`plugins/ly/skills/board/SKILL.md`, listed in `INDEX.md`) tells agents to use only `lyx board` and never read `_board/*.json`.
- **Docs.**
  The board entry in `docs/overview.md` is updated.

## Design decisions

- **New filename.**
  A pre-upgrade binary never reads or rewrites `board.json`, so it cannot silently drop the new fields.
- **Migration on load, persisted on the first write.**
  Reads write nothing, since read paths such as `battencli` and `ideengine` have no output config to render with.
  Tasks migrate to tier 1, deferred tasks and notes to tier 3, every entry to `type: feature`.
  A note id colliding with a task id gets a fresh id.
  A slug present in both legacy files refuses the load and touches nothing.
  The legacy files stay on disk.
- **Legacy done-mark fold.**
  A long-lived pre-upgrade driver can still call `MarkTaskDone` on `tasks.json` after the deploy.
  While a legacy file exists, each load applies every legacy `done` mark once, tracked in `legacy_done`.
  It marks only existing slugs and nothing else, so it never reverts a status the new binary set later, never marks a new entry that reuses a legacy slug, and never resurrects a pruned entry.
  `retire-legacy` runs one last fold, drops `legacy_done`, deletes both legacy files and their swaplocks, and refuses when none exist.
- **Lock.**
  The write lock stays `board.lock`, so old and new writers serialize.
- **JSON envelopes** keep the `task` key, so the discussion stencil and agents see no change.
- **Done and abandoned.**
  Done entries render under Done whatever their tier; `abandoned` entries render in their tier with a status tag and survive `prune`.
- **Live state** in the README is the stored `status` only, because producer or PR state would churn a weft:main commit per step.
- **Design-doc bodies** are not rewritten with hard breaks; bodies use headings and lists, which the skill says.

## To check by hand

- No production deploy was run.
  After `update-plugins.sh`, confirm the first `lyx board` write migrates the real `_board`, renders the new README and syncs `board.json` to weft:main.
- `retire-legacy` has only run in tests; run it once no pre-upgrade driver is alive.
- A pre-upgrade process writing after migration renders the old README layout until the next new-binary write.
  A pre-upgrade write other than a done mark is not folded.
- The batten bullet in `docs/overview.md` still says a board task's `type` must be `loom` or empty, which describes the old recipe field.
- Migrated entries are all tier 1/3 and `feature`; re-tiering, re-typing and rewriting briefs is the operator's job through `lyx board` after deploy.
- `docs/benchmarks/board-performance.md` still names `tasks.json`, as recorded.
