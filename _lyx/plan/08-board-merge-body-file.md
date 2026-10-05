# Card 8 — lyx board merge --body-file

**Edit:**
- `internal/boardcli#storeVerbs`
- `internal/boardcli/doc.go`
- `internal/boardcli/verbs_test.go`
- `plugins/ly/skills/board/SKILL.md`

**Uses:**
- `internal/boardcli#applyBodyFile`

**Intent:**
`merge` takes `--body-file <path>` with `upsert`'s semantics (D6), through `applyBodyFile` on the decoded `upsert` object: the file's content becomes `upsert.body`, `-` reads stdin, an `upsert` that already has `body` is refused, and a `-` payload with `--body-file -` is refused.
The stdin check runs before the payload's decode error is reported, as in `upsert`.
The `Long` help documents the flag in `upsert`'s wording.

Tests, integration-tagged: `merge --body-file f` sets the merged entry's body to the file's bytes; `--body-file -` reads stdin; `body` in both the payload and the flag is refused and writes nothing.

`doc.go` and `SKILL.md` list `--body-file` on `merge` beside `upsert`.

**ImpactSummary:** `lyx board merge` with the new flag only; payload-only merges are unchanged.

**Commit:** 8: lyx board merge --body-file
