---
format: 5
approved: true
language: go
---

# Plan: every strand reed places gets a window of its own

`lyx reed add` with no `--anchor` lands a strand in its own window, and the pane-era placement that split strands into the shell window under a parent is retired from reed's vocabulary, layout code, config, bindings, help texts, docs and tests, with the #673 workaround in `CLAUDE.md`.
The removal runs from the outside in, so every card builds: the resize watch loop first, then the stacked layout with its pins and hook, then the zoom bracket and pane buttons, then the config keys nothing reads any more, then the anchors themselves with their refusal and read-time remap, then shuttle's flag, then the move that upgrades a live pane an older binary left in the shell window, and last the docs.

## Card Index

1 — retire-resize-watch-loop — delete the per-worktree resize watch loop and its re-apply; the daemon keeps name repair
2 — retire-shell-window-stack — delete render's layout, reed's shell-window apply, pins, hook array and attach chain; boot and attach unpin `window-resized`
3 — retire-zoom-and-pane-buttons — delete the zoom bracket, the strand pane buttons and click, and unbind Alt+z
4 — retire-layout-config-keys — retire `collapsed_rows`, `min_full_rows` and `selvage` through `RetiredKeys`
5 — retire-pane-era-anchors — retire `below-parent` and `top`: remap on read, refuse on write, `reed add` defaults to `own-window`
6 — shuttle-run-anchor-vocabulary — `lyx shuttle run --anchor` accepts `own-window|hidden` and refuses the rest
7 — move-shell-window-strands — break a strand pane left in the shell window out into its own window at the reconcile head
8 — sweep-docs-and-sandbox — rewrite the package docs, overview, sandbox suites and drop the #673 workaround

## Shared Decisions

- No PATTERN entry changes; the Selvage and placement rules are module-local and live in `internal/reedengine/doc.go`.
- `render` stays a pure leaf: its decode remap imports only `encoding/json`.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
! grep -rIlE 'AnchorBelowParent|AnchorTop' --include=*.go --include=*.md --include=*.yaml --exclude-dir=benchmarks internal cmd tools docs
! grep -n '#673' CLAUDE.md
! grep -rIlE -e 'below-parent|anchor: top' -e '--anchor top' --include=*.go --include=*.md --include=*.yaml internal cmd tools docs CLAUDE.md | grep -vE '^(internal/reedengine/render/types\.go|internal/reedengine/strand\.go|internal/reedengine/doc\.go|internal/(reedengine|reedengine/render|reedcli|shuttlecli)/[a-z_]+_test\.go|tools/sandbox/SANDBOX-REED(-WATCH)?-SUITE\.md|docs/benchmarks/.*)$'
```
