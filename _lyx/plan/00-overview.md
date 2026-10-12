---
format: 5
approved: true
language: go
---

# Plan: operator-followups

Five small operator follow-ups in one task: the runs overview marks a pause-covered run as paused with its minutes (D1); a held or dead advisor no longer defers the chair's held-turn notice (D2); reed's paste-submit loads its buffer from stdin so a text of any length pastes (D3); `lyx attach` and `lyx prime` become top-level shortcuts declared in one embedded file (D4, D5); and the loom status strand takes at most an eighth of the free height (D6).

## Card Index

1 — overview-pause-marker — the runs overview reads the pause record once per poll and shows a covered running, batten-paused or starting run as `paused <minutes>`
2 — hold-deferral-held-advisor — the chair's hold-deferral predicate skips an advisor whose session reads asking, idle-stalled, idle-done or dead before its up line
3 — paste-stdin-buffer — paste-submit fills its buffer with `load-buffer -` from stdin in one invocation, so a text of any length pastes verbatim
4 — shortcutcli-package — the new shortcutcli package: the embedded shortcut file, word routing and fresh-root dispatch of each argv
5 — mount-shortcuts — newRoot mounts the shortcuts, the module guards skip them, a cmd/lyx test checks the file, and the docs name them
6 — loom-status-eighth — the loom status strand's share drops from a quarter to an eighth of the free height

## Shared Decisions

- Every test change extends an existing table test (`TestRenderOverview`, `TestOverviewSource_ReadsThePrimesRunsFromDisk`, `TestEngine_Run_ChairHoldDeferral`, `TestPasteSubmitArgv`, `TestPasteSubmit`, `TestPasteSubmit_DeliversTextAndLeavesTheBufferStackAlone`, `TestStatusStrandAddSpec`); the new top-level tests are only those of the new package and the one shortcut test in `cmd/lyx`, each budgeted on its card.
- The long-paste test is tier `tmux`; every other test this plan touches is tier 1.

## verify:

go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./internal/battencli/... ./internal/seatengine/... ./internal/reedengine/... ./internal/shortcutcli/... ./internal/loomcli/... ./cmd/lyx/...
go test -tags integration ./internal/battencli/... ./internal/seatengine/... ./internal/reedengine/... ./internal/shortcutcli/... ./internal/loomcli/... ./cmd/lyx/...
