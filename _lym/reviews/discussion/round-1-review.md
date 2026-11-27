---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:156
    summary: D10's 120-second lock wait goes through internal/lock, whose bounded acquire polls every 10 ms, so a wait past a minute breaks PATTERN-daemon-wakeups (at most one wake a second, backing off while idle)
    fix: "In D10, either bound the lock wait to under a minute (the rule exempts a bounded wait that ends within a minute) and restate Q14 in the support log, or state that the wait polls from one second doubling to a named ceiling and names the new loop for the daemon-wakeups list. Add a D10 acceptance criterion for the chosen wake rate."
  - id: F2
    severity: BLOCKING
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:119
    summary: The parent wait ends at the parent's live KWin match, which exists before the parent's own placement ran, and the lock grants no order, so a child can anchor on the desktop where KWin first opened its parent and record that wrong place for good
    fix: "In D8, make the anchor the parent's placed desktop: the wait ends only when the parent's places entry holds a desktop written by an apply or record-only run after the entry's current process id, and past the bound the parent's remembered desktop, then the current one, applies as now. Change D8's acceptance criterion from 'a parent window recorded during the wait' to 'a parent whose placement is recorded during the wait', and add a case where the child takes the lock before its parent's child."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:176
    summary: lym machine start runs every beam start of a hub before it opens any window, so batten's reopens open run windows before their orch's window for as long as the beam starts take, and on a first start past 20 s they anchor to the current desktop
    fix: "In D12, state the real order (runBeamStarts, then openWindows) and either have lym machine start open the orch windows before the beam starts resume runs, or size D8's parent wait to cover a hub's start and state the first-start fallback as a bound. Drop 'parents first' unless the order is changed to make it true."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:93
    summary: D6 copies arrange-to-tiles.sh's zone fill but drops its panel clamp, which releases each bottom-row window whose tile runs under the panel and moves it with kdotool, so lym's bottom-row windows sit under the panel on a 2-row grid
    fix: "In D6, decide the panel case: clamp the bottom row as arrange-to-tiles.sh does (and add kdotool, or the KWin call that replaces it, to D1's checks), or state the overlap as a bound. Correct the Notes line that describes arrange-to-tiles.sh as only filtering and calling manage, and add a D6 acceptance criterion for a tile under the panel."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:136
    summary: D9 does not say how concurrent writers keep each other's entries, since pre-wait records are written outside the lock while post-apply rewrites renumber and drop entries, nor whether the pre-wait write keeps the entry's remembered desktop and order key
    fix: "In D9, state that every write of the places file is one state.UpdateJSON read-modify-write that changes only its own window's fields (on an apply, the renumbered desktop's entries too), and that the pre-wait write replaces only the process id and title, keeping the desktop and order key. Add a D9 acceptance criterion where two writers interleave and neither loses an entry."
  - id: F6
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:139
    summary: A window that overflows from its full remembered desktop is recorded on the desktop it landed on, so one session on a smaller screen loses its remembered place for good, against the Goal's 'returns to the desktop and zone order it last held'
    fix: "In D9, say what an overflow placement records: either keep the remembered desktop and order key and record the overflow desktop apart from them, so the next placement with room returns the window there, or state the loss as a bound in D9 and D11. Add a D8 or D9 acceptance criterion for it."
  - id: F7
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:185
    summary: The KWin behaviour D3, D4 and D6 rest on is inferred from headers and never observed, because the operator's scripts only touch the current desktop; nothing pins building a grid and calling manage on a non-current desktop, moving a window to the target desktop and the tiling output, kscreen output names matching KWin's, or how the rootTile probe tells a two-argument rootTile apart
    fix: "Run each behaviour on Neve and record it in Constraints with the command and its observed output: rootTile(output, desktop) with split, relativeGeometry and manage on a desktop that is not current; setting a window's desktops and sending it to another output (window.output is read-only, so name the call); kscreen-doctor -j output names against workspace.screens names; and the probe's mechanism and result. Add the desktop and output move to D3's apply step."
  - id: F8
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md:47
    summary: D2 leaves its process-id and caption facts to a later contract test, but both can be observed now; on Neve each lym Konsole is already its own process whose pid is the KWin window's, and its caption reads '<lym title> — Konsole', which makes the --separate rationale unestablished and the rejected alternative's 'caption changes with the shell' false
    fix: "In D2's Rationale, record the observed result (pgrep -a konsole against kdotool getwindowpid and getwindowname on Neve, with the date) in place of 'the contract test … pins that they match'. State why --separate is still needed, or drop it from D1, D2 and the D2 acceptance criterion. In support-log.md, correct the 'Match the KWin window by caption' rejected alternative."
  - id: F9
    severity: MEDIUM
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:29
    summary: D1 counts the rootTile probe among the checks whose failure keeps today's Konsole command line, but the probe runs in the query script in the detached child after Konsole has started with --separate
    fix: "In D1, split the checks into pre-launch ones (session variables, launcher, tools on PATH), which decide the command line, and the post-launch rootTile probe, whose failure leaves the --separate window unplaced with a warning. Update the D1 and D2 acceptance criteria to match."
  - id: F10
    severity: MEDIUM
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md:155
    summary: D10's window wait and D8's parent live-match wait both need KWin queries, which are D3 script runs of up to 5 s each, yet they run outside the lock that serializes the KWin steps and claim a poll rate of four a second
    fix: "In D10, name the signal each wait uses: either a D3 query run outside the lock (state that read-only queries may run unlocked, and give the poll interval as the gap between runs) or another signal that needs no script. Make D8's parent wait read the same signal."
  - id: F11
    severity: MEDIUM
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:156
    summary: D10's per-user lock has no named home; PATTERN-lym-files gives lock paths only through lock under a checkout's .lym, and a lock under one hub's .lym does not serialize the 'start every hub' placements D10's rationale needs
    fix: "In D10, name where the per-user lock lives (for example under the lymtemp per-user root or beside the per-user machine file) and which lymdirs or lock accessor names its path, consistent with PATTERN-lym-files."
  - id: F12
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:54
    summary: A run that times out, or a child that runproc.End kills mid-apply, leaves its KWin script loaded and possibly the target desktop's tiles released, since D3 unloads only after the end marker and the record states no unload or recovery on those paths
    fix: "In D3, state that the script is unloaded by plugin name on every exit, the timeout included, and in D10 how a child ended mid-apply leaves the desktop (for example by building the new tiles before releasing the old ones, or by naming the half-built desktop as a bound). Add a D3 acceptance criterion that a timed-out run unloads its script."
  - id: F13
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:175
    summary: Moving window_command into the child loses it whenever the child exits early (lock or window timeout, unsupported probe), where today lym reed window always runs it, and lym machine start starts running it only when placement applies; mode off is not said to count as applying
    fix: "In D12, state that once the child is handed a window it runs window_command on every exit path, that lym machine start's windows gain window_command (always, or only under placement), and which side mode off falls on. Add the matching D12 acceptance cases."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md:223
    summary: Under D12, window_command can run with no window_wait_ms wait after the verb returns, but lym reed window's help ('runs it next, after the configured wait') and the desktop template's window_wait_ms comment still describe the old behaviour, and the Review criterion covers only the new keys
    fix: "Extend the Review acceptance bullet to cover lym reed window's Long help and the desktop template's window_command and window_wait_ms comments describing when window_command runs under placement."
  - id: F15
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md:139
    summary: Renumbering a desktop's present windows 0..k-1 collides with the kept keys of windows not yet reopened, and D6's tie rule does not say where the reopening window goes
    fix: "In D9, make a renumber keep absent entries' relative order (renumber present windows into the gaps, or shift absent keys past them), or in D6 say which side of a tie the placed window takes."
  - id: F16
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:65
    summary: D3's rationale says journalctl --since has second granularity, but --since '@<epoch>.5' and '<date> HH:MM:SS.250' both parse on Neve
    fix: "In D3's Rationale, drop the second-granularity reason and justify the run id and end marker by back-to-back and concurrent runs alone."
  - id: F17
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md:97
    summary: D6's rationale says counting matches arrange-to-tiles.sh, but that script counts a window whose desktops include the desktop, while D6 counts only windows on exactly that one desktop
    fix: "In D6, either count windows whose desktops include the target, as the script does, or keep 'exactly' and reword the rationale so it no longer claims to match the script."
---

### [BLOCKING] D10's 120-second lock wait breaks PATTERN-daemon-wakeups

**Location:** `_lym/discussion/decision-record.md:156`

**Issue:** D10 takes the per-user lock "through `internal/lock`, waiting at most 120 seconds".
The only bounded acquire there is `lock.AcquireWriteLockWithin`, which calls `TryLockContext` with `boundedWaitRetryDelay`, set to `10 * time.Millisecond` (`internal/lock/lock.go:17,62-78`).
`PATTERN-daemon-wakeups` exempts only "a bounded wait that ends within a minute", so a 120-second wait falls under it: at most one wake a second, backing off while idle.
That is a listed constraint, so the finding is BLOCKING.

### [BLOCKING] The parent anchor is read before the parent's window is placed

**Location:** `_lym/discussion/decision-record.md:119`

**Issue:** D9 records each window's process id and title before any wait, but its desktop only after the apply under the lock.
D8's wait ends at the parent's live D2 match, which exists as soon as the parent's Konsole maps, on the desktop KWin opened it on.
D10's lock grants no order, so a child can take it before its parent's child, anchor on the parent's unplaced desktop and record that place.
Later reopens return to the recorded place, so the error is permanent.
All three lenses that ran the case found it: interactions F1, walkthrough F1 and bound-breaker 1.
It defeats `next` in the fresh-machine "start every hub" flow, the case the entry exists for, and D8's acceptance criterion assumes the opposite.

### [MEDIUM] lym machine start opens orch windows after all beam starts

**Location:** `_lym/discussion/decision-record.md:176`

**Issue:** `startHub` runs `runBeamStarts` and only then `openWindows` (`internal/machinecli/start.go:253-254`).
`beam start --all` brings up the dobbies, whose batten reopens run `lym reed window` from then on (`internal/battencli/arm.go:884-922`).
Run windows therefore open before any orch window of their hub, by as long as the hub's beam starts take.
On a first start with no remembered parent, a wait past 20 s anchors run windows to the current desktop.
Existing-paths F2 and interactions F7 report the same defect.

### [MEDIUM] The panel clamp of arrange-to-tiles.sh is dropped

**Location:** `_lym/discussion/decision-record.md:93`

**Issue:** `~/.local/bin/arrange-to-tiles.sh` releases each zone whose bottom edge runs under the panel and moves its window with `kdotool`.
Its comment says KWin's tile layout always fills the whole screen, panel included.
The walkthrough lens saw the clamp fire in the Neve journal today.
D6 and the Notes describe only the filter and `manage` steps, so a 2-row grid's bottom row ends up under the panel.
Bound-breaker 3, walkthrough F2 and my own reading of the script all found it.

### [MEDIUM] Concurrent writes to the places file are not specified

**Location:** `_lym/discussion/decision-record.md:136`

**Issue:** Many children write pre-wait records outside the lock while an apply renumbers and drops entries.
`internal/state/doc.go:11-14` says `UpdateJSON` cannot serialise a writer that computes its payload outside the file's lock.
The record says only "written through `internal/state`".
It also never says whether the pre-wait write keeps the remembered desktop and order key that D8 reads, which bound-breaker 8 raised too.

### [MEDIUM] Overflow overwrites the remembered desktop

**Location:** `_lym/discussion/decision-record.md:139`

**Issue:** Take 5 windows remembered on B2 that reopen while the laptop screen gives B2 a capacity of 3.
The 4th and 5th overflow to B3.
D9's post-apply rule records them on B3, because their desktop changed.
Back on DP-2 they stay on B3, against the Goal and against D11's "rearranges nothing on its own".

### [MEDIUM] KWin behaviour on non-current desktops and outputs is not pinned

**Location:** `_lym/discussion/decision-record.md:185`

**Issue:** Constraints cite KWin's headers for the API.
Both operator scripts work only on `workspace.currentDesktop` and `workspace.activeScreen`, so neither shows these calls working on a desktop or output that is not current.
The design rests on these operations:

- building a grid on a desktop that is not current, and calling `manage` there;
- moving the placed window onto the target desktop and the tiling output (walkthrough F5 found no step that does it);
- `kscreen-doctor` output names equalling KWin's output names;
- the `rootTile` arity probe (bound-breaker 4: QJSEngine ignores surplus arguments, so the probe as stated may always pass).

The rubric's unpinned-tool-fact rule applies.
Each fact can be observed on Neve now.

### [MEDIUM] D2's pid and caption facts are left to a later test

**Location:** `_lym/discussion/decision-record.md:47`

**Issue:** D2 says the D3 contract test "pins" that the started pid equals the KWin window's pid, which defers a fact that is observable now.
On Neve today, `pgrep -a konsole` against `kdotool getwindowpid`/`getwindowname` shows each lym window as its own Konsole process.
Its pid is the KWin window's pid, without `--separate`, and its caption is `<lym title> — Konsole`, for example `ORCH [ly:P0] root — Konsole` at pid 3913.
So D2's facts hold, but the reason given for `--separate` is not established.
The rejected alternative's claim that the caption "changes with the shell" is also contradicted.
Bound-breaker 6 and walkthrough F3 cover the same ground.

### [MEDIUM] The rootTile probe cannot decide the command line

**Location:** `_lym/discussion/decision-record.md:29`

**Issue:** D1 says a failing check keeps "the Konsole command line … today's", and it lists the probe among the checks.
The probe runs in the query script, in the detached child, after Konsole has started (D3 step 1, D10).
The D1 and D2 acceptance criteria assert both.

### [MEDIUM] The waits need KWin queries outside the lock

**Location:** `_lym/discussion/decision-record.md:155`

**Issue:** Only a D3 query run can see a window "in KWin": load the script, run it, read the journal until the end marker, for up to 5 s.
D10 runs that wait outside the lock that "serializes the KWin steps", at "four times a second".
D8's live-match wait has the same gap, since it re-reads only the places file.

### [MEDIUM] The per-user lock has no home

**Location:** `_lym/discussion/decision-record.md:156`

**Issue:** `lock` forms only tracked-file lock paths under a checkout's `.lym/locks/` (`internal/lock/tracked.go:19-29`).
`lymdirs` names only hub and worktree `.lym` homes.
A lock that serializes placements across hubs needs a named per-user location, and the record names none.

### [MEDIUM] Timeout and kill leave the script loaded and the desktop half-built

**Location:** `_lym/discussion/decision-record.md:54`

**Issue:** D3 unloads the script "only then", after the end marker.
A timeout path has no unload.
A child registered in `internal/runproc` can be ended mid-apply by `runproc.End`, which sends SIGTERM and then SIGKILL after 5 s.
An apply that released the tiles first, as `set-tiling-grid.sh` does, leaves the desktop's windows floating.

### [MEDIUM] window_command can be lost, or gained, depending on placement

**Location:** `_lym/discussion/decision-record.md:175`

**Issue:** A child that exits on a lock or window timeout, or on an unsupported probe, never runs `window_command`.
Today `lym reed window` always runs it (`internal/reedcli/window.go:129-135`).
`lym machine start` never runs it today, and it would gain it only when placement applies.
Whether mode `off` counts as applying is unstated.
Bound-breaker 9, interactions F5 and walkthrough F6 report the same defect.

### [LOW] Help and template text still describe the in-verb wait

**Location:** `_lym/discussion/decision-record.md:223`

**Issue:** Two texts still describe the old behaviour: `internal/reedcli/window.go:75-79` ("runs it next, after the configured wait") and `internal/desktophook/template.yaml:2` ("so the command sees the new window").
Under D12, `window_command` can run in the child with no wait.

### [LOW] A renumber collides with the keys of windows not yet reopened

**Location:** `_lym/discussion/decision-record.md:139`

**Issue:** Present windows renumbered 0..k-1 can share keys with absent windows that kept theirs.
D6's tie rule does not place the reopening window.

### [LOW] The journal --since granularity claim is false

**Location:** `_lym/discussion/decision-record.md:65`

**Issue:** On Neve, `journalctl --since "@<epoch>.5"` and `--since "<date> HH:MM:SS.250"` both parse without error.
The design does not depend on the claim, but its stated reason is wrong.
Bound-breaker 5, the existing-paths note and my own check agree.

### [LOW] D6's count does not match arrange-to-tiles.sh

**Location:** `_lym/discussion/decision-record.md:97`

**Issue:** The script's `onDesktop` loop counts a window whose `desktops` include the desktop.
D6 counts only windows whose desktops are exactly that desktop, yet its rationale claims to match the script.

## Rejected

- Bound-breaker claim 1, that the checks never test `DBUS_SESSION_BUS_ADDRESS`: a missing bus fails `qdbus6`, and D1 already makes that a logged warning, so no window open breaks.
- Bound-breaker claim 15, that the 120 s lock wait is short for a burst: the lens filed nothing, and F1 already governs the wait's length.
- Existing-paths' note on `checkTaskSetting` falling back to land-notice values for an unlisted key: plan-level wiring of a new key, and enumerating it is not this rubric's subject.

## Focus verdicts

- Open risks, project-level override and VS Code windows: not a defect.
  The board refuses `run_settings` on a project, and dropping a requirement the operator decided, or the windows the entry's title names, is a question about what the task is for.
  That is the parent's to settle.
  No `Decision for kde-window-placement:` note exists on the board (`lym board list`), and the record has no `Added after Discussion` entry.
- D1 and D12's premises about today's code: `lym reed window` (through batten's Open-Window row and its reopen) and `lym machine start` are the only production terminal-window paths.
  The reed server keeps the starter's KDE variables, scrubbing only agent identity (`internal/reedengine/lifecycle.go:461-469`).
  D7's description of `LookupWithSource`, which never reads the machine layer, matches `internal/dobbyengine/tasksetting.go`.
  The launcher does not return a process id today, and D2 changes that.
- Leanness and placement: Decisions and Constraints hold no rejected alternatives or recap that belongs in the support log.
  D2's `konsole --help-all` quote is evidence, not deliberation.
