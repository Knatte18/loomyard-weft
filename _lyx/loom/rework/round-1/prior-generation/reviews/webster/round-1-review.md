---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: tools/sandbox/SANDBOX-REED-SUITE.md:342
    summary: The sandbox suite still tells the operator to look for the VIEW button on line 1, which card 8 replaced with the shell button.
    fix: In tools/sandbox/SANDBOX-REED-SUITE.md line 342, replace "show the `VIEW` button on line 1" with "show the `shell` button on line 1". Then grep tools/ for any other check of the VIEW label.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/battencli/cli.go:20
    summary: The package doc still says the overview pane's one write is the content-rows report, but card 3 removed that report from the pane.
    fix: In internal/battencli/cli.go line 20, change the sentence to "It only reads the run records and the status files, without their locks, so it never blocks or slows a run." and drop the clause about the content-rows report.
  - id: F3
    severity: LOW
    class: design
    location: internal/reedengine/strand.go:675
    summary: Engine.SetContentRows has no production caller left, because card 3 removed the only pane processes that reported a line count. Its paragraph in doc.go still presents it as the live reporting path.
    fix: Delete Engine.SetContentRows from internal/reedengine/strand.go together with its tests, and remove the matching paragraph from internal/reedengine/doc.go, the entry in the zoom-bracket call-site list and the Strand.ContentRows handling that only it feeds. If the content size kind is kept for a later caller, instead rewrite the doc.go paragraph to say no pane process reports today.
  - id: F4
    severity: LOW
    class: design
    location: internal/reedengine/apply.go:158
    summary: applyOpts.SkipWhenBoxEquals and its guard in applyLayoutLockedOpts are dead in production after card 4 moved the box-equality check into reapplyLayout. Only TestApplyLayoutLockedOpts_SkipWhenBoxEquals still uses the field.
    fix: Remove the SkipWhenBoxEquals field and its early-return guard from internal/reedengine/apply.go, update the doc comment of applyLayoutLockedOpts, and delete TestApplyLayoutLockedOpts_SkipWhenBoxEquals in internal/reedengine/apply_test.go. Lower the internal/reedengine row in cmd/lyx/testdata/test-budget.yaml by one.
---

### [LOW] Sandbox suite still names the VIEW button

**Location:** tools/sandbox/SANDBOX-REED-SUITE.md:342
**Issue:** Card 8 replaces the VIEW button with the `shell` button, and its test asserts line 1 holds no ` VIEW `. This suite step still says the attached client must show `VIEW`, so an operator following it would mark a correct bar as a `FAIL`. The call site was missed.

### [LOW] Stale content-rows sentence in the batten package doc

**Location:** internal/battencli/cli.go:20
**Issue:** The overview pane no longer builds a reed engine or a report (see the diff of internal/battencli/overview.go). The package doc still claims a content-rows write that reed confines to the runs strand, which now contradicts the `overviewCmd` doc comment.

### [LOW] SetContentRows is left with no caller

**Location:** internal/reedengine/strand.go:675
**Issue:** `statuspane.ReportFromEnv` and both panes' `Report` wiring are gone, and no production code calls `SetContentRows`. The `doc.go` paragraph still calls it the one operation through which a content strand's pane process reports its line count. The code-quality rule is to delete what is unused. The plan does not name this removal, so the finding is raised against the diff's leftover rather than a plan miss.

### [LOW] Box-equality option of the layout apply is dead

**Location:** internal/reedengine/apply.go:158
**Issue:** `reapplyLayout` now compares the live box itself before entering the zoom bracket and calls `applyLayoutLockedOpts` with `SkipFocus` only. No production caller sets `SkipWhenBoxEquals`, but the field, its guard and a test of that guard remain. Card 4's intent says the apply runs "with no box guard", which leaves the option unused.

## Checked and not a defect

- Placement, anchor and focus decisions read `listSessionWindows` and `listSessionPaneWindows`, and no code sorts or renumbers windows.
- `isOwnWindow` covers the overview. Only placement, the name, the unseen mark, the one-per-session refusal and the focus rules ask `isOverview`.
- `relevantWindow` and the attach selection fall back from the newest live agent to the overview and never name the shell. With neither present they select nothing, as card 7 specifies.
- Card 5's session-from-an-older-build repair is `ensureShellWindowLocked`. It runs on attach, on every window open and after a fresh boot, and `TestEnsureShellWindow` and `TestWindowOrder_RealTmux` cover it.
- `internal/selfexec` imports only the standard library and `internal/logger`, and its leaf test pins that. The watcher execs only on a changed identity seen on two consecutive checks, and a failed exec is not retried for the same identity.
- Card 4's skip returns before `keepZoomLocked` for a box equal to `lastApplied` or `lastFailed`, and `handleWatchOutcome` re-applies on a changed box.
- Alt+Up and Alt+Down use `previous-window` and `next-window` with one conditional step past a hidden window. The `shell` button replaces VIEW in `bar.go` and `bindings.go`.
- `lyx gate test` was green for the touched packages (reedengine, render, selfexec, statuspane, loomcli, orchcli, battencli, shuttleengine) and for `cmd/lyx`, which holds the test budget check. The tmux-tier tests were not run in this review.
