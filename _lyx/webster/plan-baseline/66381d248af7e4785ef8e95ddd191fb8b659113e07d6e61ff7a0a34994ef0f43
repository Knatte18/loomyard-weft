# Card 5 — config-empty-edit-abort

**Edit:**
- `internal/configengine#EditPath`
- `internal/configengine#ErrAborted`
- `internal/configengine_test#TestEdit`
- `internal/configcli#TestDispatchHubWideBoard`
- `internal/configengine/doc.go`
- `internal/configcli/doc.go`

**Intent:**
Decision D7: an empty editor result is never saved over a layer.

`EditPath` reads the file's bytes once on entry, after `scaffoldIfMissing` and before its loop (the previous bytes of a pre-existing file), and restores those, never a later pass's.
After the editor returns nil and the file is read back, a result that is empty or whitespace-only (`strings.TrimSpace` of it is empty) is an abort, checked before the YAML parse:
a file `EditPath` scaffolded is removed, a pre-existing file is written back to its entry bytes, and `ErrAborted` is returned.
A comment-only result is not empty and saves as before.
Every caller inherits it: `Edit` on every layer, and `configcli`'s hub-wide staging copy, whose `editHubWide` already removes the staging file and reports `aborted: <path> unchanged` with no commit on `ErrAborted`; `editOne` and `editLocal` already report their `aborted: ... unchanged` refusal and run no sync.
The bound: a layer can no longer be emptied through the editor; removing the file stays the way to clear a sparse layer.
`ErrAborted`'s comment scopes the restore to the empty-result abort: on an empty or whitespace-only result a pre-existing file is put back to its previous bytes and a scaffolded file is removed.
It keeps the other abort paths described truly: a scaffolded file is removed and a pre-existing file keeps what the editor left.
`internal/configengine/doc.go`'s `Edit` paragraph and `internal/configcli/doc.go`'s editor paragraph state the empty-result abort.

Tests (both packages are at their test budget, so every test is a row):
`TestEdit`'s first row, "missing worktree file under a board is scaffolded empty", keeps its `wantSaw` of `""` but now wants an abort with the scaffolded file removed;
new rows: an empty write over a pre-existing file aborts and restores `original`;
an invalid write, then an empty one, over a pre-existing file aborts and restores `original`, not the invalid bytes the loop's second pass read, so a restore from the per-pass bytes fails it;
a whitespace-only write over a template scaffold aborts and removes the file;
a comment-only write (`# nothing yet\n`) over a pre-existing file saves.
`Edit`'s local branch returns `EditPath`'s error before its own restore, so the pre-existing-file row is its coverage, and `editOne`'s `ErrAborted` arm is already reached by `TestEditOne`'s editor-failure row;
`editLocal`'s `ErrAborted` arm stays uncovered, since this card does not change its reporting.
`TestDispatchHubWideBoard` gains "an empty editor result commits nothing and leaves the hub file": an editor writing whitespace only exits 1 naming `aborted`, zero commits, the board file equal to `seeded`, and no staging file left.

**ImpactSummary:** One branch added inside `EditPath`, reached by `Edit`'s every layer and `editHubWide`'s staging copy; callers already map `ErrAborted`, so no caller changes, and the one existing row that saved an empty scaffold flips to an abort.
