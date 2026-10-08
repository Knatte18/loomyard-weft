---
format: 5
approved: false
language: go
---

# Plan: reed-view — a reed run readable on a small screen

Give every reed session a two-line tmux bar (strand, window and run buttons, with the overview as `VIEW` and a single strand as a zoomed pane), Alt-key and mouse navigation, a `lyx reed switch` verb for run-to-run moves, spawn-order session revival, a configurable color per loom segment shown in tmux and in Claude Code's prompt bar, and a VS Code keybindings block that forwards the Alt keys; the single-line status pipeline the bar replaces is retired.
The cards follow the decision record's D1–D16 and its post-Discussion additions: vocabulary and config first, then reed's options, verb, bar, retirement, bindings, geometry, zoom and revival, then the color's path through shuttle, the spawning modules and the orch, and last the VS Code block.

## Card Index

1 — segmentcolor-package — new leaf package: loom segments, lyx's palette and its tmux colors
2 — reed-segment-color-config — reed.yaml gains one color key per segment, refused at boot when outside the palette
3 — strand-pane-options — `@strand`, `@strand_color` and `@lyx_strands` set at launch, boot and attach; segment carried on the strand
4 — reed-switch-verb — `lyx reed switch --next|--prev` steps a client through sessions in id order
5 — two-line-status-bar — two-line status formats and pane-border titles replace the single status line
6 — retire-status-line-pipeline — remove the status-line template, verb, config key and `tokenvocab`
7 — alt-key-and-mouse-bindings — Alt+z/Up/Down/Left/Right and the status-bar click bindings, pinned at boot and attach
8 — title-row-geometry — pins adjusted for the title row and run at every apply and in the attach chain
9 — zoom-bracket — a zoomed strand survives every reed step, the resize hook and the attach chain
10 — spawn-order-revival — a revived server's sessions come back in spawn order
11 — shuttle-color-command — shuttle carries the segment to reed and types the provider's color command
12 — loom-landing-segments — driver, discussion, plan, rework, describe, conflict and friction specs name their segment
13 — webster-review-segments — webster, burler, bouncer and treadle specs name their segment
14 — orch-color — the orch watcher types the orch's color at start and first in every reload
15 — vscode-keybindings-block — `lyx ide spawn` seeds a marked Alt-key block in VS Code's keybindings.json

## Shared Decisions

- **Targets are files.** Every card names files, not symbols, so no target needs a quarry glyph or a `plan:` handle; each card's Intent names the symbols it adds or changes.
- **Segment keys.** The segment constants' values (`coordinator`, `discussion`, `plan`, `webster`, `review`, `describe`, `landing`) are the reed config keys under `segment_colors`, so one vocabulary serves the launch spec, the strand record and the config.
- **Optional probe verbs.** The constraint "the capability probe gains any verb the new pins need" conflicts with D6's non-fatal pins and the Windows degrade if read as required verbs, since a required verb psmux lacks would take every reed verb down. Card 7 resolves it the way `set-hook` and `resize-pane` already are: the probe gains an optional-verb list that warns and never fails.
- **Run start time.** D12 orders pairs by "their run's start time read from the pair's own records"; no record carries a start timestamp, so card 10 reads the modification time of the pair's own run seed (`seed.json`), written once when the run is seeded.
- **Dead fields go with the pipeline.** Card 6 removes `Geometry.RepoName` with the status text, its only reader; `WorktreeName` and `HubPath` stay, since other code reads them.
- **reedcli's module doc** is `internal/reedcli/cli.go`'s header comment; the package has no `doc.go`.
- **tmux-tier tests** that need an attached client use `internal/reedengine`'s existing pty harness (`attachgeometry_integration_test.go`, `tmux && linux`), and every new reedengine integration file runs under the package's `tmuxkit.Main`.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
