# Card 7 — lyx board get --body

**Edit:**
- `internal/boardcli#storeVerbs`
- `internal/boardcli/doc.go`
- `internal/boardcli/verbs_test.go`
- `plugins/ly/skills/board/SKILL.md`

**Intent:**
`get` gains a `--body` flag (D5).
With it, `get` writes the entry's body alone, verbatim, with no envelope and nothing appended, exit 0; an empty body writes nothing, exit 0.
A target that does not exist is an error envelope naming the slug or id, exit 1, since there is no `task: null` to print.
Without the flag, `get` is unchanged, `{"task":null}` included.
The `Long` help documents the flag and its round trip with `upsert --body-file` and `merge --body-file`.

Tests, integration-tagged: a multi-line body containing a trailing newline and none prints byte-for-byte; an empty body prints nothing with exit 0; an absent slug exits 1 with an error envelope; the bytes from `get --body`, fed to `upsert --body-file`, leave the stored body unchanged.

`doc.go` and `SKILL.md` describe editing a body as a file: `get --body` out, edit, `upsert --body-file` in.

**ImpactSummary:** `lyx board get` with the new flag only; the flagless output is unchanged.

**Commit:** 7: lyx board get --body
