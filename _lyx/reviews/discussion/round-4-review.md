---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:33
    summary: "D1 rests on a zoom that stays put, but reed's window-resized pin hook (resize-pane) and every reed select-layout apply unzoom the window, and no Decision covers it"
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:92
    summary: "D7 floors a one-row top pin only in the window-resized hook, so a planned one-row top cell is applied with zero content rows and stays so until a resize fires"
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:106
    summary: "D8 Bound claims no runtime value can reach the shell as syntax, but run-shell format-expands the whole command at key press (pinned paths included) and q: escaping is only valid outside quotes"
  - id: F4
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:91
    summary: "D7 treats every readback except off as a title row at the top, but a bottom readback puts the title row in the Selvage cell"
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:196
    summary: "D15 lists the skip cases and a failed write but not a file that cannot be read, so whether an unreadable keybindings.json skips or fails the spawn is left open"
  - id: F6
    severity: NIT
    class: decision
    location: _lyx/discussion/decision-record.md:205
    summary: "D15 Bound allows one temporary file beside the target but does not say its name is unique per run, and concurrent spawns can share a directory"
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:145
    summary: "D12 has the teller supply a spawn-order key per entry although reedengine takes list order and never reads the key"
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md:64
    summary: "Review rounds records the parent change, round 1 and round 2 but no round 3 entry although round 3 has run"
---

### [BLOCKING] Zoom does not survive reed's own resize and apply paths

**Location:** `_lyx/discussion/decision-record.md:33` (D1), `:84-95` (D7), `:263-268` (acceptance criteria)

**Issue:**
D1 makes a zoomed pane the way to show one strand, and D4, D8 and D9 light buttons from the zoom state.
Reed already issues two things that unzoom a window, and the record covers neither.
First, the `window-resized` hook that D7 keeps holds `resize-pane -y` pins (`resizePinHookArgvs`, `internal/reedengine/windowsize.go`).
The Selvage band is always pinned, so the hook exists in every session with a band.
Second, `applyLayoutLockedOpts` (`internal/reedengine/apply.go`) and the attach chain (`AttachArgv`, `internal/reedengine/attach.go`) issue `select-layout`, and the watchdog's re-apply goes through the same function.

Checked on tmux 3.6 with a throwaway server:
`resize-pane -t %2 -y 3` on a window with a zoomed pane sets `window_zoomed_flag` to 0;
`select-layout` on a zoomed window sets it to 0;
a `window-resized` hook is fired on a zoomed window (`resize-window -x`), and with `resize-pane -t %2 -y 3` as the hook body the window ends unzoomed.
So any terminal resize, which is routine in the narrow VS Code terminal this task targets, drops the operator out of the zoomed strand back to the overview.
Every re-attach, `add` and `resume` does the same.
The pins also cannot size a zoomed window's cells, since tmux rescales the saved layout proportionally on unzoom, so D7's per-cell-height criterion has no defined meaning while zoomed.
The plan writer cannot settle this from the record: it needs a choice among guarding the hook and the apply paths, restoring the zoom after them, or accepting the unzoom.

**Fix:**
Add a Decision, with its bound, that says what a resize and a reed layout apply do to a zoomed window.
For example: the hook entries and the apply paths leave a zoomed window alone, and the unzoom path (`VIEW`, Alt+z) triggers the layout and pins instead; or they unzoom, re-apply and re-zoom the previously zoomed pane.
Add acceptance criteria that a window resize, a re-attach and a spawn keep the zoomed strand zoomed (or the opposite, if the accepted behavior is a return to the overview), and that the layout is correct after the next unzoom.

### [MEDIUM] D7's one-row floor works only when a resize fires

**Location:** `_lyx/discussion/decision-record.md:92`, `:268`

**Issue:**
D7 floors the row-0 pin at one content row, so a planned one-row top cell ends one row taller.
The pin is a `window-resized` hook entry and fires only on a window size change.
The cell itself comes from the layout string, which D7 leaves unchanged.
Checked on tmux 3.6: with `pane-border-status top`, `select-layout` with a one-row top cell is accepted as written and the pane reads `pane_height` 0, and no `window-resized` hook fires for it.
`resize-pane -y 1` afterwards gives a two-row cell, taking the row from the cell below, as D7 states.
So after boot, attach and every reed apply, the top pane has zero content rows until the operator resizes the terminal, and the acceptance criterion's "ends one row taller, with one content row" holds only after that resize.
The one-row case needs `collapsed_rows: 1` or a clamp on a short window, so it is rare, but D7 spells it out as a guarantee.

**Fix:**
State where the floor is applied.
Either the planned top cell is two rows whenever the title row is present, which is a layout change and contradicts "the layout planner is unchanged", or the apply path issues the corrective `resize-pane` itself and not only through the hook.
Reword the acceptance criterion to name the moment it is checked.

### [MEDIUM] D8's Bound misses format expansion of the pinned tokens

**Location:** `_lyx/discussion/decision-record.md:103-106`

**Issue:**
The Bound says the client name is the one value tmux expands at key press and that no runtime value can end a token or reach the shell as syntax.
`run-shell` format-expands its whole command at key press, not only `#{q:client_name}`.
Checked on tmux 3.6: `run-shell "printf '%s\n' '/home/u/c#S/x #{session_name} ##'"` printed `/home/u/csess/x sess #`.
A `#` in the pinned `lyx`, tmux or socket path is therefore rewritten, and `#(cmd)` in one would run `cmd` in the server's shell.
`escapeStatusText` already exists for this reason for the old status text.
The record does not say the pinned tokens' `#` is doubled, so the Bound is false as written.

A second gap: `q:` escapes with backslashes (`a'b c` becomes `a\'b\ c`), which is valid only outside quotes, while `internal/shell` tokens are single-quoted on POSIX.
The record does not say that `#{q:client_name}` stands as its own unquoted word, so a plan writer can put it inside a quoted token, where a client name with a quote ends the token.

**Fix:**
Amend D8's Decision or Bound: every pinned token has `#` doubled after shell quoting, and `#{q:client_name}` is emitted as a separate unquoted word.
Add an acceptance row that a pinned path containing `#` reaches `lyx reed switch` unchanged.
D11's binary path is the same string, so it needs the same sentence.

### [LOW] D7 keys the title-row subtraction on `off`, not on `top`

**Location:** `_lyx/discussion/decision-record.md:91`

**Issue:**
D7 says a readback of `off` or a failed readback means no title row, which leaves every other value meaning a title row at the top.
D6 sets `top` window-scoped, but a refused set (the accepted Windows degrade) leaves whatever the inherited value is, and reed's server loads the operator's own tmux configuration.
Checked on tmux 3.6: with `pane-border-status bottom` the title row sits in the last cell (a 7-row cell reads `pane_height` 6), not the first.
Subtracting a row from the row-0 pin then shortens a cell that has no title row, and the Selvage band's pin is one row too long.

**Fix:**
Key the subtraction on a readback of exactly `top`, and say that `off`, `bottom`, an empty value and a failed readback all mean no row-0 title row.

### [LOW] D15 leaves an unreadable file open

**Location:** `_lyx/discussion/decision-record.md:196`, `:203`

**Issue:**
D15 lists the skip cases for a scan that fails and for a failed write, and says any skip still launches.
A `keybindings.json` that exists but cannot be read (permission denied, a directory in its place) is neither.
Whether that skips or fails `lyx ide spawn`, which today returns an error before launching, is left to the plan writer.

**Fix:**
Add the unreadable file to the skip cases, with its reason, and a matching clause in the acceptance criterion.

### [NIT] D15 does not say the temporary file name is unique

**Location:** `_lyx/discussion/decision-record.md:202`, `:205`

**Issue:**
The Bound names "the one temporary file beside it, which never outlives the run".
Two `lyx ide spawn` runs started together share the target directory, and a fixed temporary name lets one run rename or remove the other's file.
The result is still one of the two correct contents, but the Bound's "never outlives the run" is then not guaranteed.

**Fix:**
Say the temporary file has a name unique to the run.

### [LOW] D12's spawn-order key has no reader

**Location:** `_lyx/discussion/decision-record.md:139`, `:145`

**Issue:**
D12 has the teller sort the worktrees into one ordered list and hand `reedengine` that list, and says `reedengine` only sequences it.
Each entry also carries a spawn-order key, which nothing on the `reedengine` side reads: the booter finds its position by name and the predecessors are the entries before it.
A plan writer can implement a sort in `reedengine` from it, which contradicts "only sequences".
The same section states the "recorded and not live" test twice, as the booter's trigger and as the revive function's skip, with different wording.

**Fix:**
Drop the key from the entry, or say what `reedengine` does with it.
Say the booter's trigger and the revive function's skip are one predicate, and name the identifier the entry's `name` is matched against.

### [NIT] Support log lacks a round 3 entry

**Location:** `_lyx/discussion/support-log.md:64-80`

**Issue:**
`## Review rounds` holds the parent change, round 1 and round 2.
Round 3 has run, as the round-3 review and fixer-report files named in this round's prior-rounds list show, and its changes are in the record.
Without its entry the log no longer says what was already decided about D7, D12 and D15.

**Fix:**
Append the round 3 entry, in the form of the round 1 and 2 entries.

## Focus verdicts

- **D7, the pane at window row 0.**
  Settled in the record and not a defect.
  D7 defines it as the first cell of the layout string actually applied, in the physical pane order the string is built from, and says a pin's emission order does not decide it.
  The code supports that: `toRenderInputs` already holds `paneOrder`, and `Rules` resequences the stack placements by it.
  `FixedHeightPins` documents that it takes no `paneOrder`, so the plan writer has to change that contract, and the acceptance criterion with strand panes in a non-intended physical order covers it.
  Selvage is always placed last, so it is never the row-0 cell.
- **D7 against D6.**
  D7 does key on the effective `pane-border-status`, as it does for `#{status}`, and a failed readback keeps every pin at its cell height.
  The remaining defect is the value mapping, F4.
- **D7 at the lower bound.**
  D7 states the floor and the taller cell, and the pin hook produces it after a resize (tmux 3.6: `resize-pane -y 1` gives a two-row cell).
  The apply path does not, F2.
- **D12, teller input and the recorded-and-not-live test.**
  The predecessor's revive function decides for that predecessor, under its own op lock, and the booter decides only from its own state, so `reedengine` reads no other worktree's state, `PATTERN-told-geometry` holds and the teller's revive function supplies what the deciding side needs.
  The name and the revive function are enough; the spawn-order key is surplus, F7.
- **D15, atomic write.**
  A failed write is a reported skip, the identical-content no-op comes before the read-only check, and the Bound covers the temporary file and its removal.
  The gaps are F5 and F6.
- **D8 quoting.**
  The socket path is pinned and only `#{q:client_name}` is expanded at key press, so the focus's `#{socket_path}` premise does not apply to the record as written.
  The Bound is still incomplete, F3.
