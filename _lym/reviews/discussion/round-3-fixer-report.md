# Round 3 fixer-report — discussion

Every finding's premise held against the Codex `rust-v0.164.0` source, read through `gh api`, and against `internal/shedadapters`.

## Changes

- F1 (MEDIUM), fixed differently.
  D5's Warn, D6's Bound and the Prepare acceptance bullet now name the main checkout's `.codex/config.toml` for a linked worktree, whether or not the worktree holds its own.
  D6's Bound states the hook merge (`merge_root_checkout_project_hooks`) and that rules load per layer (`ConfigLayerEntry::config_folder`, `load_exec_policy`), so the `.codex/rules/` refusal still checks the worktree alone.
  Not made as proposed: the review's Warn fires whenever the main checkout holds the file.
  The source merges the main checkout's hooks only into a project layer that exists, and `discover_project_layers` in `codex-rs/config/src/loader/mod.rs` skips a directory with no `.codex/` folder (`continue` when the metadata is not a directory).
  The Warn therefore fires when the worktree's root holds a `.codex/` directory, with or without a `config.toml`, since a missing `config.toml` still sets `load_root_checkout_hooks: true`.
- F2 (LOW), fixed as proposed.
  A new Constraints bullet records that `writable_roots` seeds the workspace roots under `-s` with legacy syntax and no selected profile, and that a `-c` key reaches that config through the `SessionFlags` layer.
  The parent's marking stays.
  D6's launch-line paragraph and the first Open risks bullet now leave the build one check: that the sandbox lets the write through the `.lym` and `_lym` links.
- F3 (LOW), fixed as proposed.
  D2 and the probe-and-resume acceptance bullet: a record without the assistant fields reads as a round with no assistant whose chair runs under the bare round, whatever the config says.
- F4 (LOW), fixed as proposed.
  D1 places the marker and outputs under the anchor's `.lym` at the review path's subpath (`PATTERN-durable-vs-ephemeral-state`), with only the Go copy under `_lym`.
  D6's Bound drops "and markers" from the tracked content.
- F5 (LOW), fixed as proposed.
  Support log Turn 3 credits the sandbox fact to the weft writable root (D6) and the record keeping to the Go copy (D2).
  The support log gains a Round 3 entry.

## Self-check

- fix-consistency, LOW: D6 called the main checkout's file "committed" while D5 and the acceptance bullet warn on any file there. Fixed as proposed: D6 now says "committed or not".
- fix-claims, LOW: `build_cli_overrides_layer` was cited in `loader/mod.rs` but is defined in `codex-rs/config/src/overrides.rs`. Fixed as proposed: the bullet cites both files.
- fix-proposal, LOW: under `-s`, seeding also needs no selected profile, and a system config, `requirements.toml` or project `.codex/config.toml` can select one. Fixed as proposed: the Constraints bullet names what selects a profile and what happens then.
- fix-proposal, LOW: once "and markers" was dropped, D6's Bound no longer said the root reaches the untracked markers. Fixed as proposed: D6's Bound now says the root reaches every seat's markers and reports under `.lym`, and that a self-written `done` marker would send the chair to an ungated report.
- fix-way-forward: no findings.

## Disputed

None.

## Deferred

None.
