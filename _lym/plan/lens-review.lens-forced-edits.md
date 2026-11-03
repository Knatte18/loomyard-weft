```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. I traced each change to the callers, tests, docs and registrations it forces, and reported each one no card lists.

- **LOW — Card 1 (D3), `UpsertPayloadKeys` gains `discussed`.**
  - The `lym board upsert` help lists every accepted payload key, including the stored-value-only keys `beam` and `front`/`on_land` (`internal/boardcli/cli.go:489-511`, inside `internal/boardcli#Command`).
  - Card 1 does not list `internal/boardcli#Command`, so the help will omit the new accepted key `discussed`.
  - Fix: card 1 edits `internal/boardcli#Command` (the upsert `Long`) to add a `"discussed" bool — lym's flag; only lym writes it` line. Card 5 already edits `Command`, so it could carry the line there instead.
- **LOW — Card 7 (D6), `approve_seam` gains `discussion`.**
  - The comment at `internal/shedrecipe/entries_gate.go:3` says `bouncerEntry` resolves `"commit_seam"/"approve_seam"` against `Env.CommitPlan/Env.CommitDiscussion/Env.ApprovePlan`. With the change it leaves out `Env.ApproveDiscussion`.
  - No card lists `internal/shedrecipe/entries_gate.go`.
  - The reverse problem: card 7 lists `internal/shedrecipe/doc.go` for "the same correction where it describes the key", but that file never mentions `approve_seam` or `ApprovePlan` (grep found nothing). That target edits nothing; the stale description is in `entries_gate.go`.
- **LOW — Card 7 (D6), `loomcli/doc.go` target.**
  - `internal/loomcli/doc.go` sits under `**Edit:**`, but the card only adds one sentence to it. Card 8 and card 9 list the same file under `**Extends:**`.
  - Fix: move it to `**Extends:**` on card 7 too.
- **LOW — Card 4 (D5), the claimed Bolt files.**
  - The `BoardDir` comment at `internal/fabricengine/junctionnames.go:117` says the board dir holds "board.json, its overview and design files". After card 4 it also holds the replay-carried `decision-<slug>.md` records.
  - No card lists it.
- **LOW — Card 2 (D4), the write sequence.**
  - The `# Concurrency and sync` section of `internal/boardengine/doc.go` states the write order: "lock → load → mutate → save board.json → render → write files".
  - Card 2 inserts the record move/removal step between save and render, but its Intent names only the `# Rename` and `# Decision record` sections. Card 2 already edits `doc.go`, so naming this section in the Intent is enough.

Tool calls beyond the inherited context:
1. Grepped the callers of `applyDesignFile` and `UpsertPayloadKeys`, the `on_land` mentions, and resolved the Uses glyphs. I needed to confirm the re-sign callers and the glyph spellings.
2. Read `internal/boardcli/cli.go:440-515` and `help_test.go:40-80`. I needed to see whether the upsert help enumerates the payload keys.
3. Grepped shedrecipe tests and docs for `approve_seam`/`ApprovePlan`, loomcli tests for the decision verb text, and the board skill for payload keys. I needed to find text that card 7, 8 and 9 make stale.
4. Grepped `decisionDeps{` literals, read `payload_test.go:105-135`, and checked the shedrecipe and loomcli `doc.go` for `approve_seam`. I needed to confirm every builder of the deps that cards 8 and 9 extend, and where the key is documented.
5. Grepped `intentionallyNil`/`NilSeams` usage and the `shedrecipe.Env{` literal builders. I needed to check which seam lists a new Env field must join.
