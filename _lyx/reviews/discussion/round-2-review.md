---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:D5 (Attach bullet), D10
    summary: D5 puts the attach-time window selection in AttachArgv's chain, which the code drops to the bare argv whenever the strands' window holds fewer than two panes, the state D3 and D4 make normal
    fix: In D5's Attach bullet, replace "AttachArgv's chain selects the relevant window" with a rule that the selection is a select-window on the relevant window appended to every argv AttachArgv returns, the bare argv of every degrade path included, and that it is not subject to the layout chain's guards (fewer than two panes, no placed strand, a window-size or status readback failure). Add to D10 that its rename and move run in the attach pre-flight ahead of those guards. Add an acceptance criterion that attach selects the newest live agent's window when Selvage is the only pane in the shell window.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D10
    summary: D10 logs every failed rename or move at Warn and retries it at each open and attach, so a refusal that persists, such as move-window -a on psmux, logs a Warn per open and attach, against D2's Debug for psmux refusals and PATTERN-log-levels' once-per-episode rule
    fix: In D10, replace "a failed rename or move is logged at Warn" with "a failed rename or move is logged at Warn once per session and at Debug while it persists, and at Debug on psmux, as D2 does", and keep the retry at the next open or attach.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:D5
    summary: D5 says the highest-index live agent window is by D2 the newest live agent, but on psmux D2's plain fallback lets a new window take the lowest free index, so the relevant window can be an older agent; D6 states the psmux caveat for its own rule, but D5 does not
    fix: Append to D5's paragraph defining the relevant window the sentence - On psmux, where D2's order is best-effort, the relevant window may be an older live agent's.
---

### [BLOCKING] Attach selection rides a chain the new window layout suppresses

**Location:** `_lyx/discussion/decision-record.md`, D5 Attach bullet, and D10's attach pre-flight.

**Issue:** D5 says `AttachArgv`'s chain selects the relevant window.
In `internal/reedengine/attach.go` the chain exists only when every layout guard passes.
`AttachArgv` returns `bareAttachArgv` when the client size is unknown (before taking the lock), when the window-size or status readback fails, when `len(live) < 2` for the strands' window, or when no placed strand is live.
D1 to D4 move the overview, the driver and the orchestrator out of the shell window, so Selvage is alone there in the common case, `len(live) < 2` holds, and every attach takes the bare path.
Implemented as the decision states, attach would never select the relevant window in the normal session, which is the Goal's second sentence and the fifth acceptance criterion.
The suppression also logs `attach chain suppressed` at Warn on every such attach, now an ordinary state.
D10's migration step "in the attach pre-flight" has the same exposure: it is skipped where the pre-flight returns early on an unknown client size.

### [LOW] D10's Warn on a failure that recurs each open and attach

**Location:** `_lyx/discussion/decision-record.md`, D10.

**Issue:** D10 retries a failed rename or move at every later window open and attach, logging each failure at Warn.
A refusal that persists, such as `move-window -a` refused on psmux, then logs one Warn per open and attach for the life of the session.
D2 logs psmux's refused `-b`/`-a` at Debug because the order there is best-effort, and `PATTERN-log-levels` (the `internal/logger` Level policy) has a repeating failure log Warn once per episode and Debug while it persists.
The race half of the focus question holds: D10 runs under the op lock, which both `OpenWindow` and `AttachArgv` take, so a concurrent open or attach waits, and a partial success (rename done, move failed) leaves a state the next check repairs.

### [LOW] D5's newest-agent premise is unqualified on psmux

**Location:** `_lyx/discussion/decision-record.md`, D5 relevant-window definition.

**Issue:** D5 defines the relevant window as the highest-index live agent window, "which by D2 is the newest live agent".
D2's psmux fallback opens with the plain `new-window`, which takes the first free index, so after an agent window ends a newer agent can sit below an older one.
D6 names this best-effort order for its own rule; D5 states its premise without it.
Off psmux, the focus question's check holds: every `new-window` insertion has an explicit target (overview `-b -t` lowest, own windows `-b -t` shell, hidden `-a -t` highest, the no-shell fallback `-a -t` highest non-hidden), so index order is start order, and D6's single conditional step reaches a visible window because hidden windows always sit last.

## Focus verdicts

- `internal/selfexec` under `PATTERN-leaf-packages`: settled, not a defect.
  The Constraints add a background section and a `leaf_enforcement_test.go` through `scankit.AssertImportAllowlist`, matching `internal/segmentcolor`.
  `internal/statuspane/doc.go`'s import list gaining `internal/selfexec` agrees with D8, which has `statuspane.Pane.Run` call the helper.
- D2's explicit targets and D5 and D6 under them: hold off psmux; the psmux gap in D5 is F3.
- D10's failure and race handling: the race is bounded by the op lock; the logging is F2, and the attach pre-flight placement is part of F1.
- D1's replace rule and D5's shell bound: agree.
  Replacing a `top` status strand removes one pane from the shell window, which stays open with Selvage, so D1 switches nothing and the viewer stays on the shell.
  The only path where that replace ends the shell window is one where Selvage's pane is already gone, so the viewer is not moved off a live shell.
