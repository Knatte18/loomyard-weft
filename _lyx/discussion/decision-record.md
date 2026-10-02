# Decision record: bolt-v2

## Goal

Bolt becomes the roadmap.
One entry store replaces `tasks.json` + `notes.json`, every entry carries a numeric `tier` and a `type`, and the README renders like `manifest/roadmap.md`: one section per tier, one `1.` item per entry.
Agents get `lyx board find`, a compact `--text` listing, payload shapes in every verb's `--help`, and a `ly:bolt` skill, so nobody falls back to jq on the JSON files.
The store migrates itself on first use by the new binary, and a pre-upgrade `lyx` still running in a long-lived process keeps working against the board without losing its writes.

## Scope

In:

- One store file `bolt.json` in the board directory, shaped `{"version": 1, "entries": [...]}`, replacing `tasks.json` and `notes.json` as the source of truth.
- Entry fields `tier` (1, 2 or 3) and `type` (closed set, see Decisions); the old recipe-carrying `type` field becomes `recipe`; `deferred` is retired into the tier.
- Migration from the legacy files, the legacy done-mark fold, and a `retire-legacy` verb that removes the legacy files.
- One verb set over the store: `upsert`, `upsert-batch`, `set-status`, `remove`, `get`, `list`, `list-full`, `merge`, `set-deps`, plus new `find`, `promote`, `prune`, `retire-legacy`, and the existing `rerender`, `sync`.
- The `notes` group and `promote-note` stay as hidden aliases onto the same store.
- README and design-doc rendering rewritten (tier sections, item shape, design-doc header, line-preserving body).
- `--text` compact listing on `list` and `find`.
- Payload shapes documented in the `Long` help of every verb that takes a JSON payload.
- A new skill `plugins/ly/skills/bolt/SKILL.md` (`ly:bolt`) telling agents how to use bolt, listed in `plugins/ly/skills/INDEX.md`.
- Callers of the old fields updated: `internal/battencli`'s recipe read moves from `Type` to `Recipe`.
- Docs: `internal/boardengine` and `internal/boardcli` package docs, the `lyx board` help text, and the board entry in `docs/overview.md`.

Out:

- Rewriting entry briefs as what/why and re-tiering or re-typing migrated entries: a content job the operator and hub do through `lyx board` after deploy.
- Moving roadmap.md's Someday into bolt and retiring `manifest/`: task `manifest-retire`.
- Rendering producer or PR on running entries.
- Any claimability refusal in `lyx loom start` or `lyx fabric add`.
- An MCP server (`lyx-mcp` is its own entry).
- Renaming the `lyx board` command, the `_board` directory, or `board.yaml`.

## Decisions

1. **Store file is `bolt.json`, a versioned object.**
   Rationale: a new filename means a pre-upgrade binary never reads or rewrites the new store, so it cannot silently drop the `tier`/`type`/`recipe` fields it does not know; the `version` key gives later schema changes a hook.
2. **Entry schema.**
   Fields: `id`, `slug`, `title`, `tier`, `type`, `recipe` (omitempty), `depends_on`, `isolated`, `brief`, `body`, `status` (omitempty), `short_name` (omitempty).
   `id` stays in the data (get/remove by id keep working) and is never rendered.
   Rationale: minimal change from the legacy record; ids are already unique across both legacy files.
3. **Tier is an integer 1–3 and nothing else in the data.**
   1 = concretized and claimable, 2 = planned next but not concretized, 3 = loose idea.
   A new entry created without `tier` gets 3.
   The section names ("Planned", "Next Up", "Someday") and their one-line meanings live in the renderer alone.
   Rationale: the operator's spec, verbatim.
4. **`type` is a closed set `feature`, `bug`, `chore`, `design`; default `feature`.**
   `design` means the deliverable is a decision or design document, not shipped code.
   An upsert with any other value is refused, naming the allowed set.
   Rationale: tier already carries "idea", so an `idea` type would duplicate the tier axis; four values cover every current entry.
5. **The recipe field is renamed `recipe`.**
   An upsert carrying a `type` that is not in the type set is refused with a message pointing recipe names at `recipe`.
   `internal/battencli`'s board read returns the entry's `Recipe`.
   Rationale: the operator asked for `type` by that name; the recipe field is set on no entry today.
6. **`deferred` is retired.**
   An upsert carrying `deferred` is refused with a message naming `tier: 3` as its replacement.
   Rationale: deferral is exactly "not claimable", which the tier now expresses.
7. **Dependency rule: an entry's dependencies have a tier less than or equal to its own.**
   Checked on every write over the projected store, including tier changes through upsert or promote, in place of the old deferred rules; dangling-dependency, isolated and cycle checks stay.
   A refusal names the offending dependency and the entry to promote or demote.
   Rationale: generalizes the old "a non-deferred task cannot depend on a deferred one", so a tier-1 entry is never blocked by something not yet concretized.
8. **Migration happens on load, persists on the first write.**
   When `bolt.json` is absent and a legacy file exists, every load builds the store in memory from the legacy files:
   tasks become tier 1, deferred tasks tier 3, notes tier 3; every migrated entry gets `type: feature`; a legacy `type` value moves to `recipe`.
   A note whose id collides with a task id gets a fresh id; a slug present in both legacy files refuses the load, naming the slug and leaving every file untouched.
   Reads persist nothing; the first locked write saves `bolt.json`, renders, and syncs.
   The legacy files stay on disk.
   Rationale: read paths built without output config (battencli, ideengine) cannot render, so only writes persist; leaving the legacy files keeps a pre-upgrade process's reads working.
9. **Legacy done-mark fold on every load.**
   While a legacy file exists beside `bolt.json`, each load reads it and, for every legacy entry with status `done` whose slug exists in the store with another status, sets that entry to `done`.
   Nothing else is folded: no new slugs, no other statuses, no field edits.
   Writes persist the folded result.
   Rationale: the only write a pre-upgrade binary still makes after deploy is a landing's `MarkTaskDone`, from a long-running Go driver started before the deploy; a done-only, present-slug fold is idempotent, needs no snapshot, and cannot resurrect a pruned entry or revert a newer status.
10. **`lyx board retire-legacy` removes the legacy files.**
    It runs one last fold, saves `bolt.json`, deletes `tasks.json`, `notes.json` and their swaplock files, renders and syncs.
    With no legacy file present it refuses, saying there is nothing to retire.
    Rationale: stale JSON beside the store misleads agents that read files directly; removal is an explicit operator step taken once no pre-upgrade driver is alive.
11. **Write lock stays `board.lock`.**
    Rationale: an old binary writing a legacy file and a new binary folding it serialize on the same lock.
12. **Verbs: one set over the store.**
    `upsert`, `upsert-batch`, `set-status`, `remove`, `get`, `list`, `list-full`, `merge`, `set-deps` keep their names and payload shapes, gaining `tier` and `type` and losing `deferred`.
    The JSON envelope keys stay `task` (get, upsert results) and the current list keys.
    Rationale: the deployed discussion stencil and agents read `{"task": ...}`; keeping the keys makes the upgrade invisible to them.
13. **`notes` group and `promote-note` become hidden aliases.**
    Every `lyx board notes <verb>` runs the top-level verb of the same name over the same store; `promote-note` runs `promote` with target tier 1.
    Both are hidden from help and their `Short` says they are aliases.
    Rationale: `manifest/HANDOFF.md` and agents still call `lyx board notes upsert`; a hidden alias keeps them working without advertising two names.
14. **`lyx board promote '{"slug": ..., "tier": N}'`.**
    `tier` is optional and defaults to the entry's tier minus one; the target must be lower than the current tier, so promoting a tier-1 entry or to an equal or higher tier is refused.
    Skipping tiers is allowed.
    Demotion goes through `upsert` with a `tier`.
    Rationale: the operator's promotion path (3 → 2 → 1, skipping allowed) as one verb; demotion is rare enough for upsert.
15. **`lyx board prune` removes every `done` entry.**
    It strips the removed slugs from every remaining entry's `depends_on`, and the render's sidecar cleanup deletes their design docs.
    It takes no payload and reports the removed slugs.
    Rationale: folds in `board-prune-done` as specified; only `done`, as the item names.
16. **`lyx board find <text>`.**
    Positional arguments joined with single spaces form one case-insensitive substring, matched against slug, title, brief and body of every entry, done included.
    Output is the `list` JSON shape restricted to matches, or the `--text` listing.
    Rationale: the hub's jq fallback searched everything.
17. **Compact listing: `--text` on `list` and `find`.**
    One line per entry, columns aligned: tier, type, slug, title, then the status in brackets when set.
    Order is the README's order.
    Rationale: `--text` is the existing lyx convention (quarry); the operator asked for tier, slug and title, and type and status are what the hub needs to scan next.
18. **README shape.**
    `# Bolt`, a one-line intro, then `## Planned`, `## Next Up`, `## Someday`, `## Done`, each followed by its one-line meaning; a tier section renders its heading and meaning even when empty, and `## Done` is omitted when no entry is done.
    Done entries render under `## Done` whatever their tier; every other entry renders under its tier.
    Within a section, order is dependency layer then id.
    Each entry is one item:
    ```
    1. **<title>** — `<slug>` · <type>[ · <status>]
       <brief>[ [design](<design-prefix><slug>.md)]
       After `<dep>`, `<dep>`.
    ```
    The design link appears only when the entry has a body; the `After` line only when it has dependencies.
    No per-entry headings, no ids, no layer letters.
    Rationale: roadmap.md's readability, which the operator named as the model.
19. **Live state is the stored `status` only.**
    Rationale: the README is committed to weft:main, so producer or PR state from run directories would churn a commit per step and pull run state into a pure renderer; the operator marked live state optional.
20. **Design doc shape.**
    `# <title>`, a metadata line (`` `<slug>` · <tier section name> · <type>[ · <status>] ``), a `Depends on:` line linking each dependency's design doc or naming it when it has none, then the body.
    Rationale: a design doc opened alone says what it belongs to.
21. **Design-doc body renders line-preserving.**
    Outside fenced code blocks, a single newline between two non-blank prose lines becomes a hard line break (trailing backslash), except when the next line starts a list item, heading, fence, table row or blockquote, or the current line is a heading or table row.
    The stored body is never modified; only the rendered file is.
    Rationale: the operator's complaint that single-newline bodies render as one paragraph; a hard break shows lines as written without turning every sentence into its own spaced paragraph.
22. **Help documents every payload.**
    Every verb taking a JSON payload carries `Long` text listing each key, its type, whether it is required, and one example, following `get`'s current shape.
    Rationale: the operator asked for `get`/`remove` explicitly; documenting every verb removes the guesswork that drove agents to raw files.
23. **Agent skill `ly:bolt`.**
    `plugins/ly/skills/bolt/SKILL.md` covers: bolt is read and written only through `lyx board`; tier and type meanings; creating, promoting, finding and listing; the payload of each common verb; never reading `_board/*.json` directly.
    Rationale: the operator asked for a skill or doc; a skill reaches every agent through the plugin cache, a doc only agents that open it.

## Constraints

- `CONSTRAINTS.md` is authoritative; the relevant invariants are the CLI/Cobra Invariant (`Short` on every command, hidden aliases included, help-tree tests), the Fabric Git Invariant's board carve-out (writes to weft:main only through `Bolt`), Hub Containment (board reached from the hub only), and Lyxdirs Single-Declarer.
- Every weft commit passes a positive-only file list; the sync must stage `bolt.json` and the legacy-file deletions.
- `*.swaplock` and `*.lock` stay gitignored; `bolt.json.swaplock` is covered by the existing pattern.
- Tests build hubs through `internal/hubforge`; no test touches the real `_board`.
- `boardengine`'s render stays a pure function; I/O stays in `RenderToDisk`.
- No change to `manifest/` beyond what `manifest-retire` owns; `manifest/roadmap.md` does not move.
- Markdown written in this task follows semantic line breaks; the generated design docs' trailing-backslash hard breaks are renderer output, not authored prose.
- Production deploy is the operator's `update-plugins.sh`; nothing in this task runs it.

## Auto-mode assumptions

- No operator answered; every decision above is a self-pick, logged in the support log's question ledger.
- Migrated notes go to tier 3, not 2: briefs mark several as parked or someday, and the operator re-tiers during the content review.
- Every migrated entry gets `type: feature`; re-typing is the operator's content job.
- New entries without `tier` get 3 even through the top-level `upsert`, which used to create claimable tasks; agents that create claimable work pass `tier: 1`, and the `ly:bolt` skill says so.
- Claimability stays a data and rendering concept; no verb outside `lyx board` checks it.
- `abandoned` entries render in their tier with the status tag, not under Done, and `prune` keeps them.

## Open risks

- A pre-upgrade process that writes after migration renders the old README layout and deletes design docs of post-migration entries through its own render sidecar; the next new-binary write restores both, but weft:main may carry the old layout for one sync in between.
- A pre-upgrade write other than a done mark (an upsert, a status other than done) is not folded and is lost; deploy-time CLI calls are all new-binary processes, so only in-memory long-running drivers can produce one.
- Running `retire-legacy` while a pre-upgrade driver is still alive makes that driver's board reads fail as not-found: its landing logs a warning instead of marking done, and an old batten prime fails to seed a child.
- Defaulting `upsert` to tier 3 changes what an agent unaware of tiers creates; the README shows it under Someday, which makes the mistake visible.

## Acceptance criteria

- `bolt.json` is the only store a new-binary write produces; no write creates `tasks.json` or `notes.json`.
- Migration:
  - A board holding only legacy files loads with tasks at tier 1, deferred tasks at tier 3, notes at tier 3, every entry typed `feature`, legacy `type` values moved to `recipe`, ids preserved.
  - A note id colliding with a task id is reassigned.
  - A slug in both legacy files refuses the load and leaves every file byte-identical.
  - A read on an unmigrated board writes nothing; the first write creates `bolt.json` and leaves the legacy files in place.
- Fold:
  - A legacy entry marked `done` after migration marks the matching store entry `done` on the next load.
  - A legacy entry whose slug is absent from the store is ignored.
  - A non-done legacy status never changes the store.
  - A pre-upgrade-shaped `SetStatus` on the legacy `tasks.json` under `board.lock` followed by any new-binary write leaves that entry `done` in `bolt.json`.
- `retire-legacy` folds, deletes both legacy files and their swaplocks, and refuses when none exist.
- Validation:
  - `tier` outside 1–3 is refused.
  - `type` outside the set is refused, naming the set and `recipe`.
  - `deferred` is refused, naming `tier: 3`.
  - A dependency on a higher-tier entry is refused on create, on demotion of the dependency, and on promotion of the dependent.
  - Dangling, isolated and cycle checks still hold.
- `promote` defaults to one tier lower, accepts a skip, and refuses an equal, higher, or below-1 target.
- `prune` removes every done entry, strips them from other entries' `depends_on`, removes their design docs on render, and reports the removed slugs.
- `find` matches case-insensitively in slug, title, brief and body, includes done entries, and returns the `list` shape; `--text` prints the compact lines.
- `list --text` prints one aligned line per entry (tier, type, slug, title, bracketed status when set) in README order.
- README golden test: sections in order with meaning lines, an empty tier still rendered, Done omitted when empty, item shape exactly as in Decision 18, no ids or layer letters, dependency-layer ordering within a section.
- Design-doc golden tests:
  - Header and metadata line.
  - `Depends on:` links.
  - Single newlines between prose lines become hard breaks.
  - Fenced code, tables, headings and list starts are left alone.
  - The stored body is unchanged.
- `get` returns `{"task": ...}` with `tier`, `type`, `recipe`; `get`/`remove` by id still work.
- `lyx board notes upsert` and `lyx board promote-note` act on the same store; both are hidden from `lyx board --help` and carry a `Short`.
- Every payload-taking verb's `--help` lists its keys, types, required flags and an example; help-tree tests pass.
- `internal/battencli` reads the child recipe from `recipe`; `internal/loomcli`'s `MarkTaskDone` marks a migrated entry done.
- `plugins/ly/skills/bolt/SKILL.md` exists and is listed in `plugins/ly/skills/INDEX.md`.
- `go test ./...` passes.

## Notes for the plan writer

- Engine lives in `internal/boardengine` (`store.go`, `board.go`, `task.go`, `render.go`, `layer.go`), CLI in `internal/boardcli/cli.go`; the two-store scaffolding is `boardCriticalSection`/`writeOp` and the cross-store slug check, all of which collapse.
- The old deferred rules in `store.go`'s `validateWrite` are what Decision 7 replaces.
- External callers: `internal/loomcli/landingdeps.go` (`MarkTaskDone` → `SetStatus`), `internal/battencli/wire.go` (`ReadBoardType` → recipe), `internal/ideengine/menu.go` (`GetTask` → title).
- The discussion stencil `contracts/stencils/loom/loom-template-discussion.md` reads `{"task": ...}` from `lyx board get`.
- `--text` precedent: `internal/quarrycli`.
- `docs/benchmarks/board-performance.md` is a measurement report naming `tasks.json`; it stays as recorded.
