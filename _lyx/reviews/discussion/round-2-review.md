---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (read-time remap and Bound), Acceptance criteria row on a leftover strand pane that is the shell window's only pane
    summary: D3's read-time remap of a retired anchor to own-window removes the only way reed finds the shell window when Selvage is missing, so the heal-beside-then-move path D3 and its acceptance row promise can split Selvage into whatever window is current and the move cannot tell which window is the shell
    fix: In D3, add a sentence after the Bound paragraph stating how the shell window is identified when no recorded Selvage pane is listed — namely the window carrying the `@lyx_strands` marker, which both the Selvage heal's window resolution and the move use in that case, never the session's current window. Extend the acceptance row on the leftover only-pane case so the operator is viewing an agent window when the up or resume runs, and Selvage still lands in the shell window beside the leftover pane. Delete the plan-writer note on the strands'-window resolver fallback, since the Decision now settles it.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D3 (move as the first step of reconcile)
    summary: D3 puts the move inside the reconcile but does not say reconcile re-lists panes and windows after it, while reconcileLocked is handed both listings by its caller, so a stale window-0 listing would still schedule the moved dead pane for the dead-pane kill D3 says it escapes
    fix: In D3, after the sentence placing the move before the dead-pane kill, add that once any pane is moved, reconcile re-lists the shell window's panes and the session's windows before it plans, so a moved pane is judged only as an own-window pane.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, the retired-anchor grep row
    summary: The grep row names below-parent, AnchorBelowParent and AnchorTop but not the retired value top that D1 also retires, so a leftover top anchor literal passes the row
    fix: "In the grep row, add the greppable forms of the retired top value, `--anchor top` and `anchor: top`, to the list of strings no `.go`, `.md` or `.yaml` file names outside the listed exceptions."
---

### [MEDIUM] The read-time remap removes the shell window's identity when Selvage is missing

**Location:** `_lyx/discussion/decision-record.md` D3 and the acceptance row on a leftover strand pane that is the shell window's only pane.

**Issue:** The strands' window is resolved by `strandWindowTarget` (`internal/reedengine/windowtarget.go:20`) from `recordedPaneIDs` (`internal/reedengine/selvagepane.go:436`): the Selvage pane first, then any recorded strand pane that is not own-window, and with neither listed, `exactSessionWindowTarget` (`=session:`), the session's current window.
`shellWindowOf` (`selvagepane.go:426`) knows the shell window only through the Selvage pane.
D3 makes a retired anchor read as `own-window` at load, so a leftover strand pane drops out of `recordedPaneIDs` (`isOwnWindow` is true for it).
In the case D3's Bound and its acceptance row single out, a leftover strand pane alone in the shell window with Selvage missing, no tier names the shell window any more.
`upLocked` and `Resume` run `ensureSelvagePaneLocked` (`internal/reedengine/lifecycle.go:699`, `:798`), which lists panes through that resolver, so Selvage is split into the session's current window, which can be an agent window the operator is viewing, not "beside" the leftover pane.
The move itself is keyed on "a bound strand pane in the shell window", and with no Selvage pane the shell window is undefined, so the record does not say what the move does in that state, or with two leftover panes and no Selvage.
The plan-writer note asks the plan writer to "check" this fallback, which leaves a design decision to the plan.
The `@lyx_strands` window option that `markStrandWindowLocked` (`internal/reedengine/paneoptions.go:57`) sets is available as the identity.

### [LOW] Reconcile does not re-list after the move

**Location:** `_lyx/discussion/decision-record.md` D3.

**Issue:** `reconcileLocked(st, live, windows)` (`internal/reedengine/reconcile.go:223`) receives both listings from its caller, and `planReconcile` builds its dead-pane kill set from `live`, window 0's panes, irrespective of a strand's anchor.
A move run as reconcile's first step changes both listings.
Unless reconcile re-lists, a dead pane just broken out still appears in the stale window-0 listing and is scheduled for `kill-pane`, the outcome D3 says the move prevents.
The existing code treats this kind of re-enumeration as load-bearing (`spawn.go`'s "Order matters" comments), so the record should state it.

### [LOW] The grep row omits the retired value `top`

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.

**Issue:** D1 retires `below-parent` and `top` and the constants `AnchorBelowParent` and `AnchorTop`.
The grep row covers three of the four, so `--anchor top` (present today in `tools/sandbox/SANDBOX-REED-SUITE.md:326`) or a `top` anchor in a YAML fixture outside the exceptions passes it.
A bare `top` is not greppable, so the row needs the qualified forms.

## Focus verdicts

- Order against reconcile: settled, not a defect.
  D3 places the move as the first step of reconcile, before the dead-pane kill, untracked reap and binding clears, and states that a dead pane is moved before the kill reaches it; F2 covers the re-listing the order depends on.
- Order against the Selvage heal and the only-pane Bound: the order is stated (heal before move in up and resume, matching `upLocked` and `Resume` today) and the Bound names the only-pane case and why `break-pane` must skip it; F1 covers the Selvage-missing identity gap that case depends on.
- Standing step versus one-shot: settled, not a defect.
  D3 states the move is a standing step of every reconcile, gated by no upgrade marker, and names the older and `.dev-bin` binaries as the reason.
- Acceptance rows: both CLI refusals of D2 have a row, each listing `below-parent`, `top`, `overview`, an unknown and an empty value; the grep row misses `top`, F3.
