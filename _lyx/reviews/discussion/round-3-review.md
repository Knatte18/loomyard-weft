---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:38
    summary: "D1 condition 3 rests on every strand pane carrying a non-empty pane_start_command, but reed pins default-command non-fatally, so a pane started without the pin has an empty comparand and the record names no outcome"
    fix: "In D1's condition 3 and its Bound, add the case of an empty #{pane_start_command}: condition 3 is skipped for that pane, so it reads live, as for an empty foreground command. Add the cause to the premise sentence that says a strand pane freezes the pinned default-command, namely that the pin is session scoped and each set-option is non-fatal (internal/reedengine/windowsize.go shellOptionArgvs and the pin routine below it, where an empty shell path or a failed set-option leaves the session unpinned, and a session booted before the pin existed has none). Record the evidence with its version, a pane split in a tmux 3.6 session with default-shell set and no default-command lists pane_start_command as empty. Add one Open risk line that such a pane keeps the #746 false-live, and one acceptance criterion that a shell-only pane with an empty start command reads live."
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:36
    summary: D1 says condition 2 needs no listing change, but condition 3 needs two tmux fields that the session-wide window listing every bound own-window pane arrives through does not carry
    fix: "After the sentence ending with no listing, reconcile or render change in D1 condition 2, add that condition 3 is the one listing change. A bound own-window pane reaches Status only through the session-wide listing (windowPaneFormat and windowPane.livePane in internal/reedengine/ownwindow.go, merged by presentPanes), which carries neither pane_current_command nor pane_start_command, so both fields join that listing and the LivePane it maps to. State that pane_start_command can hold spaces, like the pane title already the last field, so the two fields need a delimiter or an order the parser can split."
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:219
    summary: Acceptance criteria omit the negative cases of the bounds D6 and D7 state, and D3's stop-lock guarantee
    fix: "Extend the D6 criterion at the line beginning A reed add asking for a name a dead, non-hidden strand holds with a sentence that a hidden holder, a holder inside D1's launch grace and any holder while the pair is paused keep the name, and the add numbers it as today. Add to D7's compaction criterion that the withdraw clears the resume-noticed mark of a hub-pause resume line it removed. Add to D3's criterion that lyx orch stop takes start.lock, so a status restart never binds a watcher to a strand a concurrent stop removes."
---

### [MEDIUM] D1 condition 3 has no outcome for a pane with an empty start command

**Location:** `_lyx/discussion/decision-record.md:38`

**Issue:** Condition 3 compares `#{pane_current_command}` with the pane's own `#{pane_start_command}` and justifies the comparand by "a strand pane splits with no command, so tmux freezes the session's pinned `default-command`, the shell, into that pane at spawn".
That holds only while the pin is set.
`shellOptionArgvs` in `internal/reedengine/windowsize.go` yields no pin for an empty shell path, and the routine that issues the pins treats each `set-option` as non-fatal; a session booted before the pin existed has none either.
I reproduced the consequence on tmux 3.6: with `default-shell` set and no `default-command`, a pane split with no command lists `pane_current_command` as `bash` and `pane_start_command` as empty; with both pinned, it lists `/bin/bash`.
For such a pane the record gives no rule, so a plan writer either compares `bash` with the empty string and reads the pane live (the #746 failure, silently kept) or invents a rule the record never decided.
D1 already decides the analogous case for an empty foreground command (skip condition 3); the empty start command is the missing sibling.
Reading it live is the safe direction, so the fix is a stated bound, not a redesign.

### [LOW] D1 says no listing changes while condition 3 needs one

**Location:** `_lyx/discussion/decision-record.md:36`

**Issue:** Condition 2 correctly says a pane the operator moved reads present "with no listing, reconcile or render change".
Condition 3 then needs fields no existing listing holds.
`presentPanes` in `internal/reedengine/ownwindow.go` takes every bound own-window pane from the session-wide `windowPane` listing, whose `windowPaneFormat` has window id, own flag, group, pane id, dead flag, pid and title, and whose `livePane()` maps to a `LivePane` with none of the foreground or start command.
`validateDisplay` admits no anchor that binds a pane elsewhere, so no strand escapes that listing.
A plan writer reading condition 2's sentence beside condition 3 gets no sign that the listing and its parser change, and the title is already a space-bearing last field that `parseWindowPanes` splits with `SplitN`.

### [LOW] Acceptance criteria miss the bounds D6, D7 and D3 state

**Location:** `_lyx/discussion/decision-record.md:219`

**Issue:** The D6 criterion covers a dead holder reclaimed and a live holder numbered.
D6's own bounds, a hidden holder, a holder inside the launch grace and a paused pair, which exist so the reclaim does not kill or steal a name, have no criterion, and these are the negative cases of a destructive path.
D7's decision to clear the resume-noticed mark after its withdraw, and D3's guarantee that `lyx orch stop` takes `start.lock`, likewise have none.

## Rejected

- Focus 1, D5 under a hub pause: not a defect. `pauseTick` returns before `tickIdle` and `tickHandoff` run, so the only clocks that run on a request after the resume are `tickIdle`'s stale-request age (`now.Sub(req.RequestedAt)`) and `tickHandoff`'s phase timeout started at `CycleRequestedAt` (`internal/orchengine/watcher.go`); no other tick reads the request. D5 excludes `pauserecord.Record.CoverageOf` hub intervals from both, `CoverageOf` with an empty slug covers hub entries only and its live entry runs to now, and `Retention` is stated as the bound. The acceptance criterion that a request made under a pause longer than `handoff_timeout_s` runs its cycle to completion after the resume exists.
- Focus 2, serialization of `cycle-request`: not a defect. The only writer and removers in production are `RequestCycle` (`internal/orchengine/state.go`), `ClearCycleRequest` called from `requestCycle` (`internal/orchcli/refresh.go`) and from `tickIdle` twice and `removeSatisfiedRequest` in the watcher; D5 puts each, plus withdraw, under the state lock with a compare of mode and request time, and an unlocked watcher read of a truncated marker is covered by the compare and the temp-file rename.
- Focus 3, D1 condition 2 against `validateDisplay` and `presentPanes`: not a defect. Every term checks out as built: the three anchors, the retired anchors decoding as own-window (`internal/reedengine/render/types.go`), overview counting as own-window (`isOwnWindow`), a session-wide `listWindowPanes -s`, and reconcile keeping a binding the session listing holds (`planReconcile`, `internal/reedengine/reconcile.go`). The record uses no shared-window term. A pane moved to another window of the session is present as claimed; F2 is about condition 3 only.
- Focus 4, D2 against reconcile: not a defect. `planReconcile` clears an own-window binding the session listing lacks, `Up` (which `lyx orch start` runs before it reads strands), `Resume`, `launchStrandLocked` and `RemoveStrand` all reconcile, and `Status` does not, so D2's claim that the placement survives every public op but `Status` and stays visible to `lyx orch status` until a removal holds. Removal's session guard (`paneIDsInSession`, `killReplacedOwnWindowPaneLocked`) is as D2 states.
- Focus 5, D5 withdraw against the board entry's refusal: not a defect. D5 lists the departure and its reason, and the Auto-mode assumptions repeat it. The board entry's section 4 is a wish list, not an operator decision, so a recorded departure suffices.
