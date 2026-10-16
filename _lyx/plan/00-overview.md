---
format: 5
approved: false
language: go
---

# Plan: creel stage 1 fixes

Fix the eight defects the creel stage 1 audit filed (#657 to #664) so creel reaches every hand-added strand or says why it cannot, loses no mail and prints consistent output, and write the audit's scenarios C1 to C13 into the shuttle sandbox suite.
The order follows the layering creel → shuttle → reed: reed's styled capture first, Claude's placeholder and restored-prompt readings, shuttle's box reads and restoring interrupt, then creel's seam, verbs, client and daemon, and the sandbox scenarios last.

## Card Index

1 — reed-styled-capture — reed gains a styled pane capture and ReedOps carries it
2 — claude-placeholder-blank — Claude turns a styled capture into a plain one with a placeholder-only box blanked
3 — shuttle-box-reads — shuttle's idle probe, pre-write guard and landing check read the blanked capture, and the pane-only probe reports a draft
4 — claude-restored-prompt-clear — Claude declares the n-line restored-prompt clear beside its one-C-u clear
5 — shuttle-interrupt-restoring — shuttle's interrupt for creel clears a prompt its key restored
6 — creel-stall-reason — the Delivery seam's idle answer carries a stall reason and the engine journals each change
7 — creel-stall-wait — a waiting inject hears stall changes on stderr and its envelope carries the last reason
8 — creel-interrupt-clear — creel uses the restoring interrupt, a failed clear counts as sent, and no delivery runs during it
9 — inject-ttl-from-posting — each inject is charged TTL only from its own posting
10 — read-checked-write — read acknowledges nothing it could not write, and shows a returned message's mark
11 — creel-output-consistency — log, help texts, waiting verbs' sender and release match doc.go
12 — daemon-log-file — the spawned daemon's stderr goes to a per-hub log a give-up quotes
13 — daemon-log-lines — the daemon writes its start header and pid-tagged warnings to that log
14 — creel-sandbox-scenarios — C1 to C13 join the shuttle sandbox suite after S9

## Shared Decisions

### Decision: live-capture-facts

- **Decision:** the facts D1 and D3 leave to the plan were read off a live capture of a fresh Claude Code session in a private tmux server at plan time: the placeholder and prompt suggestion are drawn as SGR faint (`ESC[2m`) text after the caret and an NBSP, typed and restored text carries no attribute, the restored prompt lands in the box within the 2 s settle with the caret at its end, one `C-u` deletes only the caret's row, a `BSpace` at a row's start joins it to the row above, and n `C-u` with one `BSpace` between each pair emptied a three-row prompt.
- **Rationale:** the decision record's auto-mode assumptions name these as what the plan confirms; cards 2 and 4 pin them as fixtures and tests.
- **Applies to:** cards 2, 3, 4 and 5

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
