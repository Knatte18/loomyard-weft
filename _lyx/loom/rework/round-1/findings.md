---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/reedengine/strand.go (Engine.SetContentRows, deleted in commit 4ddcb5133)
    summary: The run deletes Engine.SetContentRows, the only way a content-sized strand reports its line count, which retires content sizing that the decision record's Scope Out keeps valid and working for an explicit caller
    fix: Restore what commit 4ddcb5133 deleted, Engine.SetContentRows in internal/reedengine/strand.go, the applyOpts.InstallPins field and its two uses in applyLayoutLockedOpts in internal/reedengine/apply.go, TestSetContentRows in internal/reedengine/strand_test.go and the SetContentRows parts of internal/reedengine/zoom_integration_test.go, for instance with git revert 4ddcb5133 resolved against the current tip. In internal/reedengine/doc.go put SetContentRows back into the zoom-bracket call-site list and restore its paragraph, replacing the line No pane process reports a content line count today... with one sentence that neither overview pane reports since D7, so the operation stays for an explicit content-sized caller. Raise the internal/reedengine row in cmd/lyx/testdata/test-budget.yaml by one, to the package's top-level test count. Keep the battencli/cli.go fix (F2) and the SkipWhenBoxEquals removal (F4) as they are.
---

### [BLOCKING] SetContentRows deletion retires content sizing that the decision record keeps

**Location:** `internal/reedengine/strand.go`, `internal/reedengine/apply.go`, `internal/reedengine/doc.go`; the change is commit `4ddcb5133` ("reedengine: fix F3 — delete SetContentRows and the InstallPins option only it fed"), made in answer to Webster-Review round 1, F3.

**Issue:** The decision record's Scope Out says: "Retiring `AnchorTop`, `AnchorBelowParent`, content sizing or the stacked layout: they stay valid for an explicit caller and keep working in the shell window."
A content-sized strand (`render.SizeContent`) takes its reported line count within its share and above its minimum, and `SetContentRows` was the one operation through which that count reached reed.
D7 drops the two overview panes' calls of it, and the acceptance criterion says only that "Neither overview pane calls `SetContentRows`".
The run then deletes the operation, so an explicit caller can still declare `SizeContent` but can never report a count, and every content strand is fixed at its minimum rows.
`doc.go` now states this ("No pane process reports a content line count today, so a content strand's ContentRows stays unreported"), but no API is left that could report one.
That is the content-sizing retirement the record puts out of scope, and the goal does not call for it.
The Webster reviewer's F3 offered keeping the operation and rewording `doc.go` as the alternative for a content kind kept for a later caller; the record decides for that alternative.

## Goal and decisions checked

Apart from F1, the diff covers the goal and D1 to D10 with no deviation found:

- D1: `render.AnchorOverview` joins the vocabulary, and a second overview strand is refused naming the holder and `lyx reed remove` (`internal/reedengine/strand.go:358`).
- D2, D3, D10: own windows open before the shell window, hidden windows after the highest-index window, and `shellWindowOf` sits in `selvagepane.go`, which closes Plan-Review round 3's carried-over `shell-window-of-outside-selvage-seam`; `ensureShellWindowLocked` renames and moves an older session's strands' window.
- D4: the loom driver (`loomcli/driverspec.go`) and the orchestrator (`orchcli/start.go`, `Focus: true`) are `own-window`; the status and runs strands are `overview` (`loomcli/bootstrap.go`, `orchcli/runs.go`).
- D5: `relevantWindow` and the attach selection, with the multi-client and failed-listing cases that Discussion-Review's carried-over `attach-select-moves-other-attached-clients` raised, now dropping the selection as D5 records.
- D6: Alt+Up and Alt+Down are `previous-window` and `next-window` with one conditional step past a hidden window; `shell` replaces VIEW.
- D7: the checklist frame, the full wait line and the "waiting on the parent: escalation from" wording (`loomcli/checklist.go:240`); both panes drop the report and the fit.
- D8: `internal/selfexec`, a leaf with its allowlist test and its `PATTERN-leaf-packages` section, called from `statuspane.Pane.Run` per poll.
- D9: `reapplyLayout` returns before `keepZoomLocked` for the last applied or last failed box; removing the dead `SkipWhenBoxEquals` option breaks no recorded decision.
- Merge-Review round 1 converged with no finding, so it carries no open risk.
- The watchdog's re-exec, which the board entry folded in from `stale-binary-daemons`, is out of scope by the record's Scope Out (left to `logging-cleanup`), and the diff does not touch it.

## Overlap

- Peer `webster-pack-last` (row 17, Merge-In; own row 22, PR-Align): its paths are under `contracts/stencils/`, `internal/websterengine/` and `.gitattributes`, and this run's diff changes none of them, so there is no overlap and no landing order to give.
- The parent's landed commits: `lyx loom peers` lists none.
- Board entry `tender` depends on this run and reuses `internal/selfexec` for its long-lived verbs, as D8 provides; no conflict.
- Shared bookkeeping: `cmd/lyx/testdata/test-budget.yaml`, the `docs/overview.md` tree line for `selfexec`, and the new `internal/selfexec` section in `pattern/PATTERN-leaf-packages.md`.
