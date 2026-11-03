# Card 5 — board-get-record

**Edit:**
- `internal/boardcli#Command`
- `internal/boardcli#storeVerbs`
- `internal/boardcli#outputGetTask`
- `internal/boardcli/doc.go`
- `internal/boardcli_test#TestCLI`

**Extends:**
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `internal/boardengine#Board.DecisionRecord`
- `internal/boardengine#Board.StoreDecisionRecord`
- `internal/boardcli_test#runCLI`
- `internal/boardcli_test#runJSON`

**Intent:**
Make `lym board get` show the flag and print the record (D10).

The `get` envelope already carries every `Task` field, so `discussed: true` shows on a discussed entry and is omitted otherwise through card 1's omitempty tag.
`outputGetTask`'s doc comment names it.

`getCmd`, inside `storeVerbs`, gains a `--record` flag: "write the entry's stored decision record alone, verbatim, instead of the JSON envelope".
Its `Long` help states the flag and its refusals beside `--body`.
The flag refuses, before any write to out:
- `--body` beside it: `--body and --record each print one field: pass one of them`.
- An absent entry: `no entry %v: check the slug, id or handle with lym board list`, as `--body` refuses one.
- An entry that is not discussed: `entry %q has no decision record: lym stores one when its run's Discussion is approved, or with `lym loom decision store` on a run past that approval`.
- A flagged entry whose record file `DecisionRecord` does not find: `entry %q is flagged discussed but has no decision-%s.md: after a rename the record may remain at the old slug's decision-<slug>.md, and the entry's next Discussion approval stores it again`.
Otherwise it writes the bytes `DecisionRecord` returns, verbatim, and exits 0.

`doc.go`'s `get` paragraph and the board skill's `get` section name `--record` and the `discussed` field.
`doc.go`'s sentence that `upsert` takes `front` and `on_land` only as the entry's stored values also names `discussed`.

A rename can report an error after it is saved, when card 2's record step fails.
The `rename` verb's help in `Command` and the board skill's rename text say so, and name both errors; in each the entry is renamed:
- `rename moved the entry but not its decision record` names the record file left at the old slug's name, which the entry's next Discussion approval stores again under the new slug;
- `rename saved the entry, but the orphaned decision record ... could not be removed` names the file at the new slug's name, to remove by hand.

Tests: the `get` verb's existing coverage is `TestCLI` (tier 1), whose steps list holds the `--body` steps (`GetBody`, `GetBodyRoundTrip`) on the fixture that sets `BOARD_SKIP_GIT` and the hub board config.
`TestCLI` gains a `GetRecord` step beside them, on that same fixture.
It upserts entries with `runCLI`, and flags one by storing a record through `boardengine.OpenHub(filepath.Dir(cwd))` and `Board.StoreDecisionRecord`, so the store honours the skip the fixture sets and spawns no sync.
- A plain `get` on the discussed entry has `discussed: true` in `task`.
- A plain `get` on an undiscussed entry has no `discussed` key.
- `--record` prints the stored bytes exactly.
- `--record` on an undiscussed entry, on an absent entry, on a flagged entry whose file was deleted, and beside `--body` each exit non-zero with their messages' ways forward.

**ImpactSummary:** One flag and one envelope field on `lym board get`, inside boardcli, and help and skill text naming the errors a saved `rename` can report. Other verbs' behaviour and callers are untouched, and the envelope only gains a key that is omitted when false.
