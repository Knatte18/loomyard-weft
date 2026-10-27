# Rename the binary lyx to lym and the lyx names with it

`lyx` collides with LyX, the LaTeX editor, whose Debian and Ubuntu package installs `/usr/bin/lyx`.
On a machine with both, whichever comes first on PATH wins, so LoomYard's binary becomes `lym`, for LoomYard Mill.

Closes #670

## What changes

- The binary: `cmd/lyx` becomes `cmd/lym`, and `tools/deploy` builds and installs `lym`.
- The hub directory suffix `-LYXHUB` becomes `-LYMHUB`.
- The `LYX_*` environment variables become `LYM_*`.
- The `_lyx/` and `.lyx/` directories become `_lym/` and `.lym/`.
- Docs, `PATTERN.md` and its background files, the `ly` plugin's stencils and skills, scripts and launchers follow.
- Dobby's `markers` directory is gone, since it held no markers.
  `dobby.log` now sits beside `state.json` and `dobby.lock` as `.lym/beam/dobby.log`.
  Board-tidy reports go to `.lym/beam/tidy/<UTC timestamp>.md`, named by their timestamp alone.
- Comment blocks the rename touched are reflowed to one sentence per line with `tools/godocreflow`.
  This changes line breaks only, never words.

## Design decisions

- The rename is one case-mapped rewrite of `lyx`, `Lyx` and `LYX`, applied to every tracked text file, with every tracked path holding those spellings moved by `git mv`.
  Package names such as `lyxcwd` and identifiers such as `LyxDirName` are renamed with the rest, so no mixed `lyx`/`lym` vocabulary is left.
- Left as they are: the spelling `LyX`, which names the editor in a deploy test that guards against a foreign binary; `go.sum`; and `docs/research/`, whose dated measurement reports record past runs under the name they ran with.
- The dobby geometry replaces `Geometry.MarkerDir` with `LogPath` and `TidyDir`, and the `MarkerDirName` constant gives way to `TidyDirName`.
- No migration code: the binary knows only the new names.
  Existing hubs are cloned afresh.
- The landing deploys nothing, since `deploy_branches` is `[]` until the cutover.

## To check or run by hand

- The cutover after landing:
  1. Clone loomyard into a plain directory and run `update-plugins.sh` there.
  2. Rename `_lyx/` to `_lym/` in one commit on Bolt and on `main-weft`, and push both.
  3. Clone the hub afresh with `lym fabric clone` as `loomyard-LYMHUB`, and start it with `lym beam start`.
  4. Set `deploy_branches` back to `["main"]`.
- Other hubs (quarry, cs, OSL-1046's) move to `lym` the same way, and machine-local state is not carried over.
- A search of tracked files outside `go.sum` and `docs/research/` for `lyx`, `Lyx` and `LYX` should find nothing.
