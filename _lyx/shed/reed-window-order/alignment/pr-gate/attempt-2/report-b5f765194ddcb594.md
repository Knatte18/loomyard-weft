---
verdict: APPROVED
---

## Attempt 1's rejection is resolved

Attempt 1 rejected the run because commit `4ddcb5133` deleted `Engine.SetContentRows` and the `InstallPins` apply option, which retired the content sizing that the decision record's Scope Out keeps valid for an explicit caller.
Plan card 10 (`533defd3a`, "10: restore-set-content-rows") restores it:

- `Engine.SetContentRows` is back in `internal/reedengine/strand.go` with its body unchanged.
  Its doc comment drops only the clause about the pane process retrying.
- `applyOpts.InstallPins` and its use in `applyLayoutLockedOpts` are back in `internal/reedengine/apply.go`.
  The `SkipWhenBoxEquals` removal (`17dc83f41`) stays.
- `TestSetContentRows` and the content-strand part of `TestZoomBracket_ZoomedStrandSurvivesEveryReedStep` are back.
- `internal/reedengine/doc.go` lists `SetContentRows` among the zoom-bracket call sites again.
  Its paragraph is back too, with one added sentence: neither overview pane reports a count, so the operation stays for an explicit content-sized caller.
- The `internal/reedengine` row in `cmd/lyx/testdata/test-budget.yaml` rises by one.

No code outside `internal/reedengine` calls `SetContentRows`, so D7 and the acceptance criterion "Neither overview pane calls `SetContentRows`" still hold.
The tip `e1659844c` also merges `main` at `c9132bd93`, which brings in only main's own commits.

## Goal and decisions checked

The rest of the diff is unchanged since attempt 1, which checked it against the goal and D1 to D10 and found no deviation:

- the `overview` anchor and the one-per-session refusal (D1)
- own windows before the shell, hidden windows after it, and the older-session check (D2, D3, D10)
- the driver and orchestrator in their own windows (D4)
- focus on attach, on a new agent window and when a viewed window ends, with the multi-client and failed-listing cases from Discussion-Review's carried-over `attach-select-moves-other-attached-clients` (D5)
- window stepping and the `shell` button (D6)
- the checklist frame and the escalation wording (D7)
- the `internal/selfexec` leaf and its `PATTERN-leaf-packages` section (D8)
- the reapply that skips the zoom bracket for an unchanged box (D9)

Plan-Review's carried-over `card10-restored-comments-stale-pane-retry` is met: the restored comments in `strand.go` and `doc.go` drop the stale clause about the pane process retrying.
The watchdog's re-exec is out of scope by the record's Scope Out, and the diff does not touch it.

## Overlap

- Peers `twb-ftr-1` and `twb-ftr-2` (row 3, Discussion-Write; own row 22) list no changed paths, and their decision record covers `tools/tokencount` run timing.
  Nothing overlaps, so there is no landing order to give.
- Parent commits: `lyx loom peers` lists none.
- Board entry `tender` depends on this run and reuses `internal/selfexec` as D8 provides; nothing conflicts.
  Board entry `orch-split` is named in the record's open risks as the place that decides `Focus` for a second prime agent; nothing conflicts.
- Shared bookkeeping files touched: `cmd/lyx/testdata/test-budget.yaml`, the `docs/overview.md` tree line for `selfexec`, and the new `internal/selfexec` section in `pattern/PATTERN-leaf-packages.md`.
