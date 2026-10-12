---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:71
    summary: D5 rests on a false premise (a dead window 0 pane is killed by reconcile at the next launch, not kept until its owner removes the strand), and D6's session-scoped reconcile would then kill dead own-window panes and end their windows, contradicting D5 and its acceptance line
    fix: In D5, replace the clause exactly as its dead pane stays in window 0 today with a decided rule for dead own-window panes, and in D6 state whether reconcile's dead-pane kill and its keep-one-dead-pane survival rule reach own windows. Either (a) exempt own-window panes from the dead-pane kill so a dead own window stays until its strand is removed (keeping D5's acceptance line), stating the kill and survival rule stay limited to window 0 as the untracked reap does; or (b) let the dead-pane kill reach own windows, run D5's client select before that kill as well, and rewrite the acceptance line An agent that dies in an own window leaves the window showing its dead pane to say the window ends at the next reed op and a client viewing it lands on window 0.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:124
    summary: D12's count bound says each opened session is ended by the pair's teardown, but the Auto-mode assumptions name the prime as a caller, and the prime has no pair teardown, so sessions opened there are bounded by nothing but the operator exiting them
    fix: In D12's Bound sentence, replace that the pair's teardown ends with a bound that holds in the prime too, for example state that in the prime an opened session lives until its provider exits or the operator runs lyx reed remove on it, that a server restart's Resume brings each back, and that this is accepted with no count cap; or restrict lyx shuttle open to pair worktrees and drop the prime from the Auto-mode assumption.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:50
    summary: D3 validates window_split only at lyx reed up, but the value is consumed at every group join and leave, which add and remove perform on an already-up session after reed.yaml may have been edited; the behavior of add or remove on an invalid value is undecided, and window_share_max has no stated valid range
    fix: In D3, state that an invalid window_split fails every reed op that reads it (up, add, remove, resume) naming the key, or that those ops fall back to width; add one sentence stating the valid range of window_share_max (for example an integer at least 0, where 0 gives every strand its own window) and what a value outside it does; extend the acceptance line at decision-record.md:166 to match.
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:46
    summary: D2's order-independence holds only when the group's live count never exceeds its declared WindowSize and window_share_max does not change between joins; a re-stepped seat added before its predecessor is removed, or a reed.yaml edit mid-group, splits one group across shared and own windows or overfills the shared window
    fix: In D2, add the rule for a join that would push a live group past its recorded WindowSize (refuse naming the group and size, or give the joiner a window to itself), and state that the shared-or-apart choice is recorded on the group by its first strand and later joiners follow that recorded choice rather than re-reading window_share_max; add an acceptance line for the over-count join.
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:140
    summary: The watchdog-daemon constraint defers to Plan a premise the code already answers, so it reads as an unresolved condition rather than a decision
    fix: Replace the conditional second sentence of the bullet with the established fact and its consequence, for example An older daemon only re-applies window 0's layout, which persists nothing and places no own-window strand, renames sessions only for strands with a present alive pane, and reaps only sessions of gone worktrees, so it leaves own-window strands untouched and no daemon restart is needed; drop the hubreconcile.Ensure clause.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:170
    summary: D12 states two observable behaviors no acceptance criterion would catch if unbuilt, the Resume of an opened session under its recorded name with the self-removal chain kept, and a pair teardown not triggering the self-removal
    fix: Extend the lyx shuttle open acceptance line with two clauses, after a server restart Resume reopens the opened session under its recorded session name in its own window and its window still closes when the session exits, and a pair teardown with an opened session live does not run its self-removal.
---

### [BLOCKING] D5's dead-pane premise is false, and D6's reconcile contradicts D5

**Location:** `_lyx/discussion/decision-record.md:71` (D5), with D6 at line 76 and the acceptance line about an agent that dies in an own window.

**Issue:** D5 says an own window keeps a dead agent's pane until the strand's owner removes it, "exactly as its dead pane stays in window 0 today".
The code says otherwise.
`planReconcile` (`internal/reedengine/reconcile.go`) puts every dead pane except Selvage's, and except one kept pane when every pane is dead, into `deadPanesToKill` and clears the bound strand's binding.
`launchStrandLocked` (`internal/reedengine/spawn.go`) runs `reconcileLocked` on every launch, and the file header says every public engine op reconciles first.
So today a dead agent's window 0 pane goes at the next reed op in the worktree, which in a busy run is the next spawn, not at its owner's removal.
D6 then says reconcile reads own-window strands through a session-scoped pane listing.
Under that listing the existing dead-pane kill sees dead own-window panes and kills them.
For the last pane of an own window that ends the window outside D5's removal path, so D5's client select never runs.
D6 also leaves the keep-one-dead-pane survival rule undefined once the listing spans windows.
D6 limits only the untracked reap to window 0, not the dead-pane kill.
The plan writer gets two incompatible readings, and one of them fails D5's own acceptance line.
D5's Rationale makes the dead window the agent's failure evidence, so this is a missing decision on the evidence the operator relies on, not a wording slip.

### [MEDIUM] D12's count bound does not hold in the prime

**Location:** `_lyx/discussion/decision-record.md:124` (D12 Bound), against the Auto-mode assumption that the orch calls `lyx shuttle open` "in the prime or any worktree".

**Issue:** The rubric's attack-surface question asks what bounds a new verb.
D12 answers "no cap on the count, since each is a numbered strand in the worktree's table that the pair's teardown ends".
The prime is never torn down as a pair, so in the prime, the expected home of the orch, nothing ends an opened session except its provider exiting or a manual remove.
D8 has `Resume` bring each one back after a server restart, so the sessions accumulate across restarts.
The stated bound is false for the prime, and no other bound is given.

### [LOW] `window_split` validation only at `up`; `window_share_max` range unstated

**Location:** `_lyx/discussion/decision-record.md:50` (D3) and the acceptance line at 166.

**Issue:** `mouseOption` and similar `reed.yaml` checks run at session boot (`internal/reedengine/lifecycle.go`).
D3's `window_split` is consumed at every join and leave, so add and remove read it on an already-up session, after reed.yaml may have been edited.
D3 leaves undecided whether those ops fail or fall back.
`window_share_max` gets a default but no valid range, so a 0 or negative value has no stated meaning.

### [LOW] D2's order-independence has two unbounded update paths

**Location:** `_lyx/discussion/decision-record.md:46` (D2 Rationale).

**Issue:** D2 refuses a size mismatch but never checks that the live count stays within the declared size.
A re-stepped seat added while its predecessor is still recorded is a third joiner in a two-seat group, and it lands in the shared window past the cap.
D2 also re-reads `window_share_max` at each join, so a config edit between two joins of one group sends the first strand to a shared window and the second to a window of its own.
Either path ends in the arrangement D2's Rationale says cannot happen.

### [LOW] The watchdog constraint leaves a determinable premise to Plan

**Location:** `_lyx/discussion/decision-record.md:140` (Constraints).

**Issue:** The focus asks whether this bullet is a decision or an unresolved premise; it is the latter.
The code answers it.
The daemon's layout tick, `reapplyLayout` (`internal/reedengine/reapply.go`), persists nothing.
`partitionByAnchor` (`internal/reedengine/render/policy.go`) places no strand whose anchor is other than top or below-parent.
`repairNames` (`internal/reedengine/namerepair.go`) skips a strand whose pane is absent or not alive.
The daemon's own reap (`internal/reedcli/watchdog.go`) acts on whole sessions of gone worktrees.
None of these is destructive to an own-window strand an older binary cannot see.
The plan writer can act on the conditional as written, but it hands over research the record can settle in one sentence.

### [LOW] D12's resume and teardown behavior lack acceptance criteria

**Location:** `_lyx/discussion/decision-record.md:170`.

**Issue:** The focus names D12's pause refusal, which is covered.
D12 also decides two other observable behaviors.
`Resume` relaunches an opened session under its recorded name and keeps the self-removal chain.
A pair teardown never triggers the self-removal.
D8's generic resume line covers neither the named-session resume nor the kept chain, and no line covers the teardown case, so either could go unbuilt with every criterion passing.

## Focus coverage

- D12: the Decision states what the verb skips (no run record, no hold notices, no turn) and bounds pause and naming. The count bound fails in the prime (F2). The removal on provider exit goes through reed's existing `remove --name --detach` (`internal/reedcli/remove.go`), runs only after the provider has exited, and teardown ends the pane's shell first, so I found no stray removal path.
- D5 and D6: the removal-path select, the `@lyx_own` window end, the op-lock ordering against an add in progress and the hand-killed pane are bounded as written. The dead-pane path is not (F1).
- D4: the hook's bound holds. Without `set-hook` the marker is never set, and the Bound limits the hook to one option on the selected window. No finding.
- D2 and D9: the size-mismatch refusal, the `WindowSize` 0 and negative rules and the refused anchor or group change hold for one config and an exact count. F4 covers the two paths outside that.
- D10: the premise holds. `internal/loomcli/driverspec.go` sets `below-parent`, and `internal/orchcli/start.go` leaves the anchor empty today, so it must become explicit as D10 says. The orch's runs strand and loom's bootstrap strand are reed adds, which the shuttle default does not reach. A missed `shuttleengine.Spec` site only puts that session in a window of its own, which is display-only and caught by the D10 acceptance line, so I raise no finding.
- Constraints, the watchdog bullet: F5.
- Relocation: no deliberation narrative in `decision-record.md` needs moving. No requirement lives only in `support-log.md`. Its rejected items, such as the order-independence of concurrent group adds, are stated in D2 and D6.
- Acceptance criteria: D3, D8, D11 and D12's pause refusal each have a failing criterion. F3 and F6 cover the gaps.
