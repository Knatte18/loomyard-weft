# Card 4 — board-load-migration

**Edit:**
- `internal/boardengine/store.go`
- `internal/boardengine/legacy.go`
- `internal/boardengine/legacy_test.go`
- `internal/boardengine/store_test.go`

**Create:**
- `internal/boardengine/migrate.go`
- `internal/boardengine/migrate_test.go`

**Uses:**
- `internal/boardengine/task.go`
- `internal/boardengine/labels.go`

**Intent:**
D12: an entry decoded from `board.json` in the old shape converts in memory on load and persists in the new shape on the next write, as the legacy `tasks.json` migration does.
Conversion never fails and never validates labels against config, so no hub needs an operator step to load; card 3's write validation refuses a later write of an entry carrying an unconfigured label, naming `board.yaml` and the label.

`migrate.go`:
- A decode-only entry shape, `Task` embedded plus `Tier *int` (`json:"tier"`) and `Type *string` (`json:"type"`), used only by `Load`; `Save` writes `Task`, so `tier` and `type` drop on the next write.
- An entry with `tier` set and `kind` empty migrates:
  - tier 1 → `task`; tier 2 and 3 → `note`;
  - `type`: `bug` → `bug`; `feature` and `chore` → `enhancement`; `design` → `enhancement` then `undecided`; an absent or unknown type → `enhancement`;
  - leading `[x]` tokens of the brief, with or without spaces between them, become labels in order after the type-derived ones and are stripped from the brief with the whitespace after them;
    a token equal to a legacy type name (`feature`, `bug`, `chore`, `design`) or to a type in the store's vocabulary, when it has one, is dropped, since the type comes from `type`;
    a repeated label keeps its first occurrence only;
  - a migrated note's `depends_on` moves into its body as a closing line `` Depends on `a`, `b`. `` (after a blank line when the body is not empty) and the list is emptied;
  - a migrated task's `depends_on` entry naming an entry that migrated to a note is removed.
    The old tier rule admitted that edge only when the dependency was done, and a done dependency already drops out of After, so nothing a reader sees is lost.
- An entry with neither `tier` nor `kind` loads as a note with empty labels, so a hand-written fixture still loads.
- An entry that already has `kind` is untouched.

`store.go`: `Load` decodes through the migration shape and runs the conversion on the whole entry list, since the dependency rewrite needs every entry's new kind.

`legacy.go`: the legacy `tasks.json`/`notes.json` path chains into this conversion: `entryFromLegacy` builds the decode shape with the tier it assigns today and type `feature`, and `migrateLegacy` returns entries that went through the same conversion, so a legacy brief's bracket prefixes become labels too.

Tests (`migrate_test.go`, `legacy_test.go`, `store_test.go`): every tier and every type, a design entry gaining `undecided`, bracket prefixes with and without spaces stripped into labels in order, a prefix naming a type dropped, a repeated prefix kept once, a note's dependencies moved into an empty and a non-empty body, a task's edge to a migrated done note removed, an already-migrated entry untouched, and a load-then-write round trip whose `board.json` carries `kind` and `labels` and no `tier` or `type`.

**ImpactSummary:** Changes only how `Store.Load` decodes `board.json` and the legacy files; entries already in the new shape load unchanged.
