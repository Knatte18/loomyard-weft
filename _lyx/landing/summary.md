# Rename lyx to lym: binary, hub suffix, env variables and state dirs

`lyx` collides with the LyX editor's `/usr/bin/lyx`, so the binary and every name derived from it become `lym`, for LoomYard Mill.

## What changed

- A case-mapping rename of `lyx`, `Lyx` and `LYX` to `lym`, `Lym` and `LYM` in every tracked text file and path, done by a one-off script that is not committed.
  This covers `cmd/lym`, `binaryName()` in `tools/deploy`, the `-LYMHUB` hub suffix, the `LYM_*` environment variables, the `_lym/` and `.lym/` directory names, the `.lym-anchor` file, the `ly` plugin's stencils and skills, docs, scripts, and the package and identifier names such as `internal/lymcwd`, `internal/lymdirs`, `internal/lymtemp` and `LymDirName`.
- Dobby's `markers` directory is gone: `Geometry.MarkerDir` is replaced by `LogPath` (`.lym/beam/dobby.log`, beside `state.json` and `dobby.lock`) and `TidyDir` (`.lym/beam/tidy/`), and a board-tidy report is named by its UTC timestamp alone, such as `20261026T151739Z.md`.
  `MarkerDirName` gives way to `TidyDirName`.
- Two `internal/gatecli` tests pin hashes computed over the renamed names; their literals are updated.
- The comment paragraphs whose words the rename changed were legacy fixed-column-wrapped, so the comment lint read their breaks as new.
  A second scripted pass rejoined each flagged paragraph and broke it only at sentence ends and semicolons, with no word changed; this is the bulk of the third commit's diff.

## Design decisions

- `LyX`, the LaTeX editor's spelling in a `tools/deploy` test that guards against a foreign binary, is left as it is.
- `go.sum` and `docs/research/` are left untouched: one holds hashes, the other dated measurement reports of past runs.
- There is no migration code: the binary knows only the new names, and existing hubs are cloned afresh at the cutover the board entry describes.
- The comment re-break touches only the paragraphs the lint flagged, not every wrapped comment in a touched file, to keep the diff to what the rename caused.

## Testing

The scoped `lyx gate test` tier0 run is green, and `lyx loom lint-comments` against the fork point reports no findings.
