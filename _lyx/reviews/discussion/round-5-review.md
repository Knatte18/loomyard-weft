---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:34
    summary: D1 does not say the live pane list is read on the unzoomed window, but a zoomed pane reports pane_top 0 so the physical order D7 builds row 0 from is wrong while zoomed, and the attach path lists panes before it can unzoom
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:34
    summary: D1 names select-layout and resize-pane -y as the only commands that unzoom, but split-window, kill-pane and select-pane without -Z unzoom too, so the re-zoom trigger is narrower than the cause
  - id: F3
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:96
    summary: The zoom bracket and the post-apply pin run are stated as reed steps, but the attach chain is a static argv tmux runs after AttachArgv returns and each hook array entry fires on its own, and no Decision says how either carries the bracket or the pins
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:226
    summary: The Constraints bullet says the teller tells reedengine the worktree list and its start times, but D12 hands reedengine only names and revive functions after the teller has sorted, so the start times have no reader
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:151
    summary: D12 has the booter re-run its whole session-ensuring step from the start yet also says the step is not re-entered, and only the revive functions are told to skip, so the booter's own re-run would meet the same trigger again
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:210
    summary: D15 says the temporary file name is unique to the run but not what happens when a file of that name already exists, nor that the run removes only a file it created
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:16
    summary: The Scope line limits session recreation to the boot that spawns the hub's tmux server, but D12 also revives on a server that stayed up and when a second booter arrives
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:155
    summary: The D12 Bound says a worktree is brought up when its state records a session and drops the not-live half of the revival predicate stated two paragraphs earlier
---

### [MEDIUM] D1 leaves the pane order read while zoomed unspecified

**Location:** `_lyx/discussion/decision-record.md:34` (D1), `:93` (D7), `:278` (acceptance criterion).
**Issue:**
D1 runs each reed step "on the unzoomed window", and D7 takes row 0 as the first cell "in the physical pane order that `select-layout` string is built from".
That order comes from `paneIDsByTop` in `internal/reedengine/apply.go`, which sorts the `list-panes` answer by `pane_top`.
Checked on tmux 3.6 with three stacked panes, the last one zoomed.
`list-panes` reported `%0 top=0`, `%1 top=11` and `%2 top=0 h=30`.
The zoomed pane reports the full window and top 0, so a stable sort by `pane_top` puts it second instead of last, and the layout string maps planned heights to the wrong panes.
Nothing in D1 says the listing follows the unzoom.
In the attach path it cannot: `AttachArgv` (`internal/reedengine/attach.go`) calls `listPanes` and plans the chained layout while the window is still zoomed, before any attach.
The acceptance criterion at line 278 would catch this only if its test zooms a pane that is not first in physical order, and it does not say so.
**Fix:**
State in D1 that the pane list and its order are read on the unzoomed window, or taken from a source that does not change under zoom.
Say what the attach path does, since it plans before it can unzoom.
Make the acceptance criterion zoom a strand that is not the top pane.

### [LOW] D1 names two unzooming commands where tmux has more

**Location:** `_lyx/discussion/decision-record.md:34` (D1 Decision), `:37` (Rationale).
**Issue:**
D1 says "`select-layout` and `resize-pane -y` unzoom a window" and triggers the re-zoom on "every reed step that issues either".
Checked on tmux 3.6, a zoomed window is also unzoomed by `split-window` without `-Z`, by `kill-pane` of another pane, and by `select-pane -t` without `-Z`.
`select-pane -T` (the title step in `namerepair.go`) and `respawn-pane` do not.
Reed issues the first three in `spawn.go`, `reconcile.go`, `strand.go`, `selvagepane.go` and the trailing focus step of `applyLayoutLockedOpts`.
The Bound ("a window that was zoomed when it started") already brackets the whole step, so most steps are covered.
A step that kills or splits and then meets an apply guard returns before any `select-layout`, and the trigger as worded does not fire for it.
**Fix:**
Word the trigger as "every reed step that changes the strand window's panes, layout or active pane", reading the zoom at step start, and list the unzooming commands as the reason.

### [LOW] D1 and D7 give no mechanism for the executions reed does not run itself

**Location:** `_lyx/discussion/decision-record.md:34` (D1), `:96` (D7).
**Issue:**
D1 describes a Go step that reads `#{window_zoomed_flag}`, acts and re-zooms.
D7 says reed "runs the pins itself right after every layout apply", the attach chain included.
Two executions are not Go steps.
The attach chain is an argv built by `AttachArgv` and run by tmux after the client attaches, so the layout apply there happens after reed has returned, and a zoom read at build time may be stale by then.
Each `window-resized` hook entry is a single tmux command that fires on its own (`resizePinHookArgvs`), so the first `resize-pane -y` unzooms and a later entry cannot know what was zoomed.
Both need the bracket and, for the chain, the pin run, expressed as tmux commands that test `#{window_zoomed_flag}` at run time.
The record states the outcome in the Bound and the acceptance criteria but not that this is how the chain and the hook carry it.
**Fix:**
Add one sentence to D1 and D7 saying the chain and every hook entry carry the unzoom, the resize and the re-zoom as run-time conditional tmux commands, and that the chain also appends the pin run after its `select-layout`.

### [LOW] Constraints says start times are told, D12 hands none

**Location:** `_lyx/discussion/decision-record.md:226`.
**Issue:**
The `PATTERN-told-geometry` bullet says "D12's ordered worktree list and its start times are told by the hub-mode teller".
D12 (line 152) has the teller sort and then tell `reedengine` entries carrying a directory name and a revive function, plus the booter's name.
The start time stays with the teller, and `reedengine` sorts nothing and reads no record.
The support log's round 4 entry records that the spawn-order key was dropped, so the Constraints bullet is the leftover.
**Fix:**
Say in the bullet that the teller tells the sorted list of names and revive functions; drop "and its start times".

### [LOW] D12's booter re-run contradicts its own no-re-entry claim

**Location:** `_lyx/discussion/decision-record.md:149`–`151`.
**Issue:**
Line 149 says the revive functions are "told to skip this step".
Line 151 says the booter "re-runs its whole session-ensuring step from the start, so ... the step is not re-entered for the booter's own session".
The step is triggered by the revival predicate on the booter's own state, and that state is unchanged until the booter creates its session, so a re-run from the start meets the trigger again.
Only the revive functions are said to carry the skip.
**Fix:**
State that the booter's re-run carries the same skip as a revive function's up, so the trigger is not evaluated twice.

### [LOW] D15 leaves a temporary-name collision unstated

**Location:** `_lyx/discussion/decision-record.md:210`–`213`.
**Issue:**
D15 gives the temporary file "a name unique to the run" and says a failed write "removes the temporary file if one was created".
It does not say what happens when a file of that name already exists, for example one left by a killed earlier run, nor that the run must not overwrite or remove a file it did not create.
The Bound says the file "never outlives the run", which a killed process cannot honour.
**Fix:**
State that the temporary file is created exclusively, a collision fails the write and is reported as a skip, and cleanup removes only a file this run created.

### [LOW] Scope narrows session recreation below D12

**Location:** `_lyx/discussion/decision-record.md:16`.
**Issue:**
Scope reads "Session recreation in spawn order on the boot that spawns the hub's tmux server".
D12, widened in round 1, runs on any revival of the booter's recorded session, a second booter arriving after the server is up included, and says it does not hold for a later-position session still live.
The Scope line is the version the plan writer reads first.
**Fix:**
Reword to "session recreation in spawn order when a worktree's recorded session is revived".

### [NIT] D12 Bound restates the predicate without "not live"

**Location:** `_lyx/discussion/decision-record.md:155`.
**Issue:**
Line 147 defines the predicate as "records a session and that session is not live".
The Bound says the step brings up worktrees "whose reed state records a session".
The revive function's own check makes the difference harmless, but the focus asked for one predicate.
**Fix:**
Say "for which the revival predicate holds" in the Bound.

## Focus verdicts

These settle the directive's other entries; none is a finding.

- **D7 floor and border-status mapping.**
  The floor is applied both from the hook and right after every apply (line 96), and the acceptance criterion at line 277 names both moments ("right after the apply and before any resize", "after a resize").
  The mapping keys on a readback of exactly `top`, with `off`, `bottom`, empty and a failed readback all meaning no title row (line 94).
  Checked on tmux 3.6 that `#{pane-border-status}` reads back `top` for a window-scoped set.
  With the default `collapsed_rows: 3`, the top strand is a pinned cell, so the row-0 adjustment applies in the common case.
  The attach-chain part of the apply is F3.
- **D8 run-shell quoting.**
  Checked on tmux 3.6 that `run-shell` format-expands its command, that `##` becomes `#`, that `##{y}` and `##(echo hi)` come out literal, and that a lone `#q` is unchanged.
  Doubling every `#` after shell quoting therefore keeps a path literal.
  `#{q:client_name}` as its own unquoted word expands to nothing when empty and leaves the other words intact, so an unknown client reaches the verb as a missing flag value and fails with a JSON error.
  The Bound states the `#` rule and the `q:` placement, and D11 carries the same rule.
  Not a defect.
- **D12 spawn-order data.**
  The data `reedengine` receives has a stated reader, the booter's position by name and the revive calls; the Constraints leftover is F4.
- **D15 unreadable file.**
  Stated: an unreadable file is skipped with its reason and left untouched, and `lyx ide spawn` still launches (lines 204 and 212).
  The temporary-name collision is F6.
