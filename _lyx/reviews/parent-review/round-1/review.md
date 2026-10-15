---
verdict: APPROVED
---

## Scope against the board entry

The decision record builds stage 1 as the `creel-core` entry asks: the per-hub daemon (D2, D3), the pool and journal with 7-day retention (D6), every verb the entry names (`post` D7, `inject` and its privilege D9, `inject --interrupt` and `interrupt` D10, `forward` D11, `read` and `inbox` D8, `log` D18), holds (D13), pause (D14), receiver rules and the removed-holder sweep (D5, D15), sender naming including `<shortname>:EXTERNAL` and `--wait-reply` (D4, D12), and delivery through shuttle gated on idle (D1, D19).
The entry's test points are in the acceptance criteria: fake panes, the tmux tier, and the idle check on a hand-added `lyx reed add --cmd claude` strand (llm tier, enabled by D17).
The entry's "the design moves into the creel package's `doc.go`" is D20.

Out of scope as the entry draws it: no module sends through creel (stage 2 `creel-notices`, stage 3 `creel-agents`), `lyx shuttle send`/`interrupt` stay where they are, and cross-hub sending and an MCP layer are left out.
D16, never typing into an orch strand in stage 1, keeps stage 1 from taking over the orch pane before `creel-notices` changes `PATTERN-orch-pane-single-writer`, so it stays inside the stage boundary rather than leaving a goal point out.

The record names its departures from the entry's wording: the socket path (D3; the entry gives its path as an example), the upper-case program-sender name (D4; the entry's `ly:batten` is an example), and a reminder backoff after creel's own unanswered reminder turn (D7).
D7 keeps the entry's rule that mail is reminded until read and changes only the repeat timing for one edge, with a recorded reason; it is a design detail for Discussion-Review, not a direction the entry rules out.

## Other board entries

- `creel-notices`, `creel-agents`: their scopes are left untouched; D20's `doc.go` carries their design for them to read, as both entries say.
- `engine-registry`, `engine-config-seams`: not running. The entry's 2026-10-15 decision says creel uses shuttle's present idle check and send and moves onto the registry when that lands; D17 and the open risks name the shared shuttle `Send`/`Interrupt` lookup and leave the adaptation to whichever lands second.

## Overlap with runs in flight

`lyx loom peers` gives `own_row` 3 and no parent commits.

- `report-timing` (row 6): report engine and shed verbs' caller label; no overlap.
- `viewer-off-shell` (row 7): reed follow state, focus rules and status-bar button; creel only reads reed's persisted state, and the follow state is a tmux option, so no behaviour or PATTERN entry overlaps.
- `kickstart-per-card` (row 12): Webster kick-start packs; no overlap.
- `orch-color-on-reexec` (row 0, no worktree yet): orch watcher's start-color typing after a re-exec; creel never types into an orch strand in stage 1 (D16) and touches no watcher code, so no overlap.

No overlap was found, so no landing order applies.
Shared bookkeeping files the record names (`cmd/lyx/testdata/test-budget.yaml`, the module-table line in `docs/overview.md`, and a new PATTERN index line for `PATTERN-creel-store`) do not count as overlaps.
