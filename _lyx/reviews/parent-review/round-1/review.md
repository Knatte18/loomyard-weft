---
verdict: APPROVED
---

Alignment at `parent-review` for `orch-context-overview`.

## Scope against the board entry

The entry asks for one orch line in the prime's batten overview: context tokens against the soft and hard thresholds, the watcher's phase and the cycle count, read from files only.
D4 draws exactly those cells, D2 reads the state file without a lock and without creating anything, and D3 takes the thresholds from the orch config.
The entry's open dependency question (does batten reading orch state cross a layer) is settled by D1: `internal/battencli` already imports `orchcli` and `orchengine`, so the entry's fallback options (a provider the orch module registers, a composed view) are not needed and are listed as rejected alternatives.
The one cell beyond the sketch, `watcher down`, marks a stale reading from a cleanly stopped watcher; it serves the entry's goal and no entry rules it out.
Scope Out keeps `lyx orch status`, the watcher, the orch state and config, the status bar (`reed-status-buttons`) and the run lines' columns untouched.

## Board entries touching this task

- `orch-split`: may later split the orch in two; it plans no overview line, and this design does not anticipate the split. No conflict.
- `orch-any-worktree`: would move an orch out of the prime; the design records the prime assumption as an open risk. No conflict.
- `reed-status-buttons` and `viewer-off-shell`: status-bar and window-follow work; this design changes neither. No conflict.
- `report-timing`: a timing report; unrelated to the orch context line. No conflict.

## Overlap with runs in flight

`lyx loom peers` lists no parent commits.

- `run-pane-timing` (row 12, at Webster): its D4 prefixes the run lines' stage cell in `internal/battencli/overview.go` with entry numbers.
  This run adds an orch line above the run lines and leaves the run lines as the parent renders them; the two change different behaviour in the same file and their decisions do not contradict.
  Its Scope Out keeps the runs overview's time columns on `FormatMinutes`, which this run does not touch.
  Landing order: `run-pane-timing` (row 12) is further along than this run (`own_row` 3), so it lands first and this run takes its changes at Merge-In.
- `role-language-skills` (row 3, tie): adds a new degrading caller to `pattern/PATTERN-config-strictness.md`; this run only cites that entry and changes no PATTERN file. No overlap.
- `glyph-audit-tool` (row 7) and `refusal-route-config-empty` (row 17): no shared behaviour, package or decision.

Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml` may be edited by this run and is already edited by `run-pane-timing`; this does not escalate.
