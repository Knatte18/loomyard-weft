---
format: 5
approved: true
language: go
---

# Plan: configurable action buttons on the reed status bar

Every reed session gains a third status line holding `follow` and a row of clickable buttons defined in `reed.yaml`'s `buttons` key, with machine additions in `desktop.yaml`'s machine layer.
The hub geometry tells reed the session's place and placeholder values lazily, reed renders the row into session options at boot and attach, and a click runs `lym reed button`, which confirms, runs, logs and reports.

## Card Index

1 — shell-detached-forms — add output discard, zero exit, command flag and a quoted-word directory change to the pane-shell seam
2 — buttons-config-keys — add the buttons key to reed.yaml and desktop.yaml with the shipped default set
3 — button-entries — decode, validate and fill button entries
4 — geometry-button-context — tell reed the session's place, placeholder values and machine buttons lazily
5 — third-status-line — add status line 3, move follow to it and re-pin the global formats on a stale build
6 — render-button-row — render the buttons into the session's row and spec options at the status pins
7 — strand-name-pane-option — set the full strand name on each strand pane
8 — reed-button-verb — add lym reed button, which confirms, runs, logs and reports a clicked button
9 — button-click-binding — bind a click on a button range to lym reed button in the run directory

## Shared Decisions

- The `buttons` value stays an uninterpreted `yaml.Node` in both config structs; only reedengine's `placeButtons` decodes it, at render, so a bad button costs only itself (D2).
- Button rendering and the click verb use `shell.ForGOOS()`, the dialect every other composed reed line uses, and its new `DiscardOutput`, `ExitSuccess`, `CommandFlag` and `ChangeDirToWord` forms (`PATTERN-shell-guard-table`).
- The status-format and row option pins stay non-fatal, like every geometry pin.
