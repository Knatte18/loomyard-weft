# Card 7 — cli-verbs

**Edit:**
- `internal/boardcli/cli.go`
- `internal/boardcli/notes_test.go`
- `internal/boardcli/promotenote_test.go`
- `internal/boardcli/cli_unit_test.go`
- `cmd/lyx/helptree_test.go`

**Create:**
- `internal/boardcli/text.go`
- `internal/boardcli/text_test.go`
- `internal/boardcli/verbs_test.go`

**Uses:**
- `internal/boardengine/board.go`
- `internal/quarrycli#newGlyphsCmd`

**Intent:**
Put the new verbs, the compact listing, and the hidden aliases on `lyx board` (Decisions 12–17).

`cli.go`:
- The verbs `upsert`, `upsert-batch`, `set-status`, `remove`, `get`, `list`, `list-full`, `merge`, `set-deps` are built by one constructor that is called twice: once for the top level and once under `notes`.
  The `notes` group is therefore an alias onto the same store by construction, and the duplicated notes command bodies are deleted.
  Payload shapes and the JSON envelope keys (`task`, `tasks`, `count`) are unchanged.
- The `notes` group and `promote-note` set `Hidden: true`, and each `Short` says it is an alias: the group's names the top-level verbs, `promote-note`'s names `promote` to tier 1.
  The aliased children keep a non-empty `Short`.
- New verbs, each with a `Short` and a `Long` in the same key/type/required/example shape the existing verbs use:
  - `promote '{"slug": ..., "tier": N}'` — `slug` required string, `tier` optional integer; unknown keys refused; outputs `{"ok":true,"task":{...}}`.
  - `prune` — no payload; outputs `{"ok":true,"removed":[...]}`.
  - `find <text>...` — at least one positional argument, joined with single spaces into the search text; outputs the `list` shape (`{"ok":true,"tasks":[...]}`).
  - `retire-legacy` — no payload; outputs `{"ok":true}`.
- `list` and `find` gain a `--text` flag, following quarry's `glyphs --text` precedent, that prints `text.go`'s compact listing instead of JSON; errors stay JSON.
- The root command's `Long` describes one store, the tiers and types, and says agents use `lyx board` rather than the `_board` JSON files; it no longer mentions `tasks.json`, `notes.json` or the notes group.
  The file comment describes the new verb set.

`text.go`: the compact listing over `[]BriefTask`, in the order given (README order, from card 6): one line per entry with the columns tier, type, slug, title aligned across lines, then `[<status>]` when the status is set.

Tests:
- `text_test.go` (untagged, pure): column alignment across slugs and types of different widths, the status bracket present only when set, input order preserved, and empty input printing nothing.
- `verbs_test.go` (`//go:build integration`, reusing `cli_test.go`'s `seedCwd`): `promote` default and explicit tier, and its refusals; `prune` reporting removed slugs; `find` JSON over slug/title/brief/body matches with a `done` entry included, and `find` with no argument refused;
  `list --text` and `find --text` printing the compact lines in README order;
  `retire-legacy` refusing on a board without legacy files and removing seeded legacy files otherwise;
  `get` by id and `remove` by id still working, and `get` returning `tier`, `type` and `recipe` under `task`.
- `notes_test.go`: `notes get` and `notes remove` reach an entry created by top-level `upsert`.
- `promotenote_test.go`: `promote-note` on a tier-1 entry succeeds and leaves it unchanged.
- `cli_unit_test.go`: `lyx board --help` lists neither `notes` nor `promote-note`, and both commands, plus every `notes` child, carry a non-empty `Short`.
- `cmd/lyx/helptree_test.go`: the board row's `wantSubs` drops `notes` and `promote-note` and adds `find`, `promote`, `prune` and `retire-legacy`.

**ImpactSummary:** `lyx board` gains four verbs and a flag; `notes` and `promote-note` keep working but leave the help listing; one cmd/lyx help-tree row changes.
