---
verdict: APPROVED
---

## Scope

The design covers every section of the `logging-cleanup` board entry and nothing more.

- `log-level-hygiene`: the inventory's six main items (D3 to D10), the board's "check the other modules' WARN calls" (D11), and the `Level policy` text (D12).
  The test noise point is D13.
- `reed-resize-pane-count`: D1 names the cause, a stale watchdog binary that predates `AnchorTop`, and D2 and D18 fix it.
  The per-tick WARN is D3.
- `run-log-archive`: D14 and D15 settle the board's open questions (a 30-day constant, verify logs archived, the prime's batten logs left out).
  The archive sits under the hub's `_board/.lyx/`, which fits the board's "under the hub's `.lyx/`, never `_lyx/`".
- `batten-land-notice`: D16 and D17.

The excluded items belong elsewhere: `gitrepo: push refused` to `run-window-bugs`, and a trace-file level to `spawn-audit-log`'s `logger-file-level` section.
No other board entry asks for any of this run's work.
`orch-split` plans a run driver that would take over the landing notice, and D16 notes it adds nothing to what that note moves.

## Overlap with runs in flight (escalated)

The scope check passes, but the design changes the same Go packages and files as both peers, so the run escalates.

- `go-driver-assistant` (Webster) has already changed `internal/battenshed/innerrun.go`, `notice.go` and `publishretry.go`.
  This run's D5 changes the levels and dedupe of `reportStuck`'s callers in `innerrun.go` (`exemptEnd`) and `publishretry.go`, and D7 changes `notice.go:413`.
  Both runs change `internal/battenshed/doc.go`, `contracts/stencils/claims_test.go` (D17) and `docs/overview.md` outside its module table (D16).
  Its tackler reads the step traces and driver log to find a stop's cause, and D5 moves many batten wait lines from Warn to Info.
- `run-window-bugs` (Plan-Burler-Review) changes `internal/shuttleengine` (`wait.go`, `run.go`, `doc.go`) and `internal/shuttleengine/claudeengine` (`claudeengine.go`, `command.go`).
  This run's D9 changes `shuttleengine`'s `readiness.go` and `send.go` and their provider-neutral types, and D10 changes `claudeengine/sessionname.go`.

Bookkeeping only, not escalated: `PATTERN.md` gains a new index line, `PATTERN-log-levels`, and `cmd/lyx/testdata/test-budget.yaml` rows may change.
The parent's landed commits list is empty.
