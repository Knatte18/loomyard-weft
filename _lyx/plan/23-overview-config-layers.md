# Card 23 — overview config layers

**Prosa:**
- `docs/overview.md`

**Intent:**
Bring `docs/overview.md`'s description of config resolution in line with the layer stack, as the record's docs constraint names.
The `_lyx/config/` row of the records table, the paragraph that says `board.yaml` is hub-wide like `fabric.yaml`, and the **config** module entry under the CLI modules each say, briefly and pointing at `internal/configengine`'s package doc for the detail: a module resolves key by key through the template, the repo default at `<BoardDir>/_lyx/config/<module>.yaml`, the worktree's sparse file and the untracked local override at `<anchor>/.lyx/config/<module>.local.yaml` (a hub-wide module skips the worktree file); `lyx config` takes `--layer default|worktree|local` and `--resolved`; a board task's `run_settings` reach the pair's local layer at batten's Seed-Child, bounded by `configreg`'s `LocalKeys` and `LocalDenied`.
The module table and the execution stack do not change.
Every inline link added resolves (`PATTERN-markdown-link-integrity`), and every new line is one sentence.
