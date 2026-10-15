---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D2 (Bound) and Acceptance criteria
    summary: D2's rationale says a resumed run re-arms follow, but the resume verb lyx loom resume, which the circling, batten halt and darn way-forwards name, is left out by D2's bound with no stated choice and no acceptance row
    fix: In D2's Bound, after the sentence that names the bound as the entry itself, add one sentence that decides the lyx loom resume case. Recommended text - "`lyx loom resume`, which wakes a parked driver without bringing reed up and never switches a client, leaves the follow state as it is; only a resume that enters through `lyx loom start` re-arms follow." Narrow the Rationale sentence to match - "The re-arm on a resume through `lyx loom start` is accepted, since it is the run's entry verb, and an operator who stopped following is moved back to it, as at its start." Then add an Acceptance criteria row directly after the row on a parked run resumed by `lyx loom start` - "A halted run woken by `lyx loom resume` with follow off leaves follow off and moves no viewer." If the author instead chooses that lyx loom resume re-arms too, name it in D2's Decision as a second caller of D2's reed operation and make the new row assert follow on and the relevant window selected.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria, row on an ending viewed window
    summary: The row says an ending viewed window moves its viewer to the relevant window, but D6 keeps ReplaceStrand's overview replacement, which moves the viewer to the new overview window even while a live agent window is the relevant window
    fix: Reword the Acceptance criteria row that begins "An ending viewed window" to - "An ending viewed window still moves its viewer as today, with follow on and off - to the relevant window, or to the new overview window when the overview strand is replaced - and leaves the follow state unchanged."
---

### [MEDIUM] `lyx loom resume` is neither in nor out of D2's re-arm by a stated choice

**Location:** `_lyx/discussion/decision-record.md` D2 (Bound) and Acceptance criteria.

**Issue:** D2's Bound names three `lyx loom start` entries that re-arm follow: a fresh run, a re-entrant start on a live run and a resume of a parked run.
Those match the code: `internal/loomcli/approve.go` and `internal/loomcli/reject.go` put `"resume": "lyx loom start"` in their envelopes, and batten's Run-Shed spawns `lyx loom start --no-attach` (`internal/battencli/wire.go`).
Each reaches `ensureStatusStrand` in `start.go`'s RunE, so D2's operation placed after it fires for all three, and the `lyx start` alias (`StartAliasCommand`) runs the same RunE.

The bound misses the second resume verb.
`lyx loom resume` (`internal/loomcli/resume.go`) wakes a halted run's parked driver, and its doc says it "never brings reed up and never switches a tmux client".
It is the way forward that `internal/battenshed/innerrun.go` names for a circling decision, that `internal/loomshed/darnwrite.go` names for Darn-Write and that `internal/reviewbudget` names after a config fix.
D2's closing sentence ("no other loom verb ... turns follow on") excludes it by implication.
D2's Rationale argues the opposite: "a resumed run is the run continuing, and an operator who stopped following is moved back to it".
A plan writer who reads the rationale can wire D2's operation into `resume` as well, and one who reads the bound leaves it out.
The two resume routes then behave differently, and no Acceptance criteria row catches either reading.
The resume behavior chosen for `lyx loom start` is pinned with follow off by the row on a parked run resumed by `lyx loom start`, so the focus question on that row is settled: it is pinned.

### [LOW] The ending-window row disagrees with D6's `ReplaceStrand`

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, the row beginning "An ending viewed window".

**Issue:** D6 keeps `ReplaceStrand`'s viewer move as it is.
In `internal/reedengine/replace.go`, an overview replacement that ends the viewed window calls `selectStrandWindowLocked(strand.PaneID)`, which selects the new overview window.
Every other replacement and `RemoveStrand` reach `selectRelevantWindowLocked` through `killStrandPanes` (`strand.go`), which selects `relevantWindow`.
With a live agent window present, `relevantWindow` is that agent window, not the overview.
A test written to the row ("moves its viewer to the relevant window") fails against an unchanged overview replace, or pushes the implementer to change `ReplaceStrand`, against D6.
The row on adding, replacing or resuming an overview strand outside the start bootstrap, which round 1 fixed, agrees with D6, since it only excepts "D6's move".

## Settled without a finding

- D4's acceptance row: the row with an overview present and a first client on the shell attaches a second client with follow on (the overview is selected and the first client moves) and with follow off (nothing is selected).
  Under today's rule in `attach.go`, which drops the selection whenever another client is attached, the follow-on half fails, so the row fails if D4 is left unimplemented.
- D3's bound names `lyx shuttle run --focus` among the `Focus` users, matching `internal/shuttlecli/run.go`, which sets `Focus: focus, KeepViewer: !focus`.
  The other `Focus` setters, `internal/orchcli/start.go` and `internal/reedcli/add.go`, are named too.
  A viewer moved by tmux keys outside reed's bindings is bounded once in D5 and referenced from D4; D3's bound is the follow state, so it needs no second statement.
- D7 decides the unresolvable-path click: from off it only sets `@lyx_follow` on and selects no window, logged at Warn as `switchBindingCommandsLocked` in `bindings.go` logs it for the session-switch keys.
- The remaining Acceptance criteria rows agree with D3, D4 and D7.
