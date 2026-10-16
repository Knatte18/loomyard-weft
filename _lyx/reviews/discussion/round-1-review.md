---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:44
    summary: D3 leaves the move's order against reconcile unstated and lists a subset of the ops that reconcile, so a dead leftover pane can be killed before it is moved
    fix: In D3, replace the parenthetical trigger list (up, resume, attach pre-flight, any add or remove) with "as the first step of the reconcile every public engine op runs, before reconcile plans its dead-pane kill, untracked reap and binding clears", and add one sentence saying a dead leftover pane is moved before that dead-pane kill can reach it.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:47
    summary: D3's bound does not cover a leftover strand pane that is the shell window's only pane, where break-pane takes the whole shell window with it
    fix: In D3, state the order of the move against the Selvage heal (heal Selvage first, so the shell window always holds Selvage or its corpse when a strand pane is broken out), and extend the Bound sentence to say that the move never breaks out the shell window's last pane, so the shell window and its markers stay in place. Add an acceptance row for a leftover strand pane that is the shell window's only pane, with Selvage missing.
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:46
    summary: D3 does not say whether the move is a standing reconcile rule or a one-shot upgrade step, so a pane an older or .dev-bin binary places in the shell window after the upgrade may stay there
    fix: In D3, replace "moving it once" in the Rationale with a Decision sentence saying the move is a standing step of every reconcile, keyed on a bound strand pane in the shell window and gated by no upgrade marker, so a strand that an older or .dev-bin binary adds to the shell window later is moved by the next op.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:111
    summary: The shuttle acceptance row pins only the below-parent refusal, but D2 gives shuttle run a new CLI refusal of every value outside own-window|hidden, overview included, which the engine accepts today
    fix: Rewrite the row to read "`lyx shuttle run --anchor` accepts `own-window|hidden`, and `below-parent`, `top`, `overview`, an unknown or an empty value exits non-zero with a message naming `own-window` and starts no run", matching the reed add row above it.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:121
    summary: The grep row names the identifier AnchorTop but not AnchorBelowParent, though D1 retires both, so render.AnchorBelowParent uses such as the one in the shuttleengine spec comment pass the grep
    fix: In the grep row, name `below-parent`, `AnchorBelowParent` and `AnchorTop`, keeping the same exception list, so the identifier and the value of both retired anchors are covered.
---

### [MEDIUM] D3's move has no stated order against reconcile, and its trigger list is a subset

**Location:** `_lyx/discussion/decision-record.md:44`

**Issue:** `internal/reedengine/reconcile.go` says every public engine op runs reconcile first, under the op lock, and `UpdateStrand` reconciles too (`reconcileApplyPersistLocked`), yet D3 lists only up, resume, attach pre-flight, add and remove.
More importantly, D3 never says whether the move runs before or after reconcile's planning.
`planReconcile` puts every dead pane in the shell window into its kill set, bound or not, apart from the kept and exempt ones.
After D3's read-time remap, the strand reads as own-window, so the binding check keeps its binding because the pane is still listed in `windows`, but the kill set still kills the pane.
If reconcile runs first, a dead leftover pane is killed and its strand stays bound to a pane that no longer exists.
That contradicts "A dead pane is moved the same way" and the acceptance row saying a dead one relaunches into its own window.

### [MEDIUM] D3's bound does not cover the shell window's last pane

**Location:** `_lyx/discussion/decision-record.md:47`

**Issue:** D3 introduces a new routing edge, `break-pane` out of the shell window.
Its bound names which panes it touches, but not the case where the leftover strand pane is the shell window's only pane: Selvage is missing, or was never re-split after a server-side loss.
Breaking out a window's only pane moves the whole window, so the shell window, which carries the strands-window marker that bindings and the status bar key on, would become the strand's "own window", and no shell window would remain.
D3 also does not order the move against the Selvage heal (`ensureSelvagePaneLocked` splits Selvage from the shell window's panes, and treats a sole-pane corpse specially), and D4's requirement that Selvage fills the shell window after D3's move depends on that order.

### [LOW] D3 leaves open whether the move is a standing rule or a one-shot

**Location:** `_lyx/discussion/decision-record.md:46`

**Issue:** "At the first op" and "moving it once" read as a one-shot upgrade step, while the bound ("bound strand panes found in the shell window") reads as a standing reconcile rule.
D6 and the Open risk already admit that older and `.dev-bin` binaries keep acting on the hub's shared server after the upgrade.
With D4 removing the shell window's layout and D6 removing its resize re-apply, a strand that such a binary adds to the shell window after a one-shot move has run would stay there, with nothing to lay it out.
The plan writer has to choose between the two readings.

### [LOW] The shuttle acceptance row is narrower than D2

**Location:** `_lyx/discussion/decision-record.md:111`

**Issue:** `internal/shuttlecli/run.go` validates no `--anchor` today and hands the value to the engine, whose `validateDisplay` accepts `overview`.
D2 makes shuttle refuse every value outside `own-window|hidden`, so `overview` and empty become CLI refusals for shuttle, where the engine accepts `overview` and D3 refuses only `below-parent` and `top`.
That split is consistent: the CLIs are narrower than the engine.
But the row pins only `below-parent`, so the new shuttle-side refusal of `overview`, `top`, unknown and empty values has no acceptance coverage, unlike the matching `reed add` row.

### [LOW] The grep row covers one retired identifier but not the other

**Location:** `_lyx/discussion/decision-record.md:121`

**Issue:** D1 retires both `below-parent` and `top` from `render.Anchor`.
The row greps the value `below-parent` and the identifier `AnchorTop`, but not the identifier `AnchorBelowParent`, which does not contain the string `below-parent`.
For example, `internal/shuttleengine/spec.go`'s comment says that a spec wanting the shell window names `render.AnchorBelowParent`.
No Decision defines or removes `AnchorTop` by name, but D1's closed-set change implies its removal, so it belongs in the row along with its sibling.
The value `top` itself cannot be grepped usefully, which is why both identifiers have to be named.

## Verdicts on the focus points with no finding

- D4's Selvage heal with the corpse as the shell window's only pane is a Decision, not a new behaviour: `ensureSelvagePaneLocked` already splits from a sole-pane corpse and kills it afterwards, because the corpse is `listStrandPanes`' only shell-window pane. D4 keeps that rule through the layout removal.
- D4 and D6 state what goes and why the shell window loses nothing: with D1 and D3, only Selvage remains there, and D6's bound limits the unpin to the shell window's window-scoped `window-resized` hook. F3 is the only gap left open.
- D5 and D6: `watchdog`, `width` and `height` staying is consistent with the Goal. `reedcli/watchdog.go`'s `enterSession` skips both the `Watch` and `WatchNames` loops on `off`, so keeping the key preserves name-repair gating. `width` and `height` still feed the unattached `new-session` size, and no Decision depends on a retired key.
- The support log holds no requirement missing from the decision record: every "fixed" lens item and every ledger pick maps to text in D1–D8, Scope, Open risks or Acceptance criteria.
