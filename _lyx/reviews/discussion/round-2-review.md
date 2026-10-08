---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:84-90
    summary: "D7 says every fixed-height pin keeps its cell height and the top pin grows by the title row, but resize-pane -y sizes the top pane content, so the pin must shrink by one and the grow direction is wrong"
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:160
    summary: "D13 refuses an out-of-palette color at config load and calls that the channel of reed's other keys, but those keys are validated at boot; load-time refusal would block down, attach, status, pair teardown and the watchdog over a color typo"
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:131-141
    summary: "D12 states the spawn-order guarantee without a bound for a revival whose later-position sessions are already live or whose predecessor up failed, and Open risks names only a hand-started server"
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:135-137
    summary: "D12 leaves open how the pre-boot step runs outside the booter's op lock when its trigger is evaluated inside that lock, and how reedengine runs another worktree's up from a told list of names and start times"
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:188
    summary: "D15 writes by temp file and rename yet promises a read-only file stays untouched; rename replaces a read-only file in a writable directory and also drops the target's mode and owner"
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:182-184
    summary: "D15 marker rules leave open a begin or end marker outside the top-level array, an orphan or repeated end marker, and the line ending of the inserted lines in a CRLF file"
  - id: F7
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:162
    summary: "D13 and D14 leave open a persisted segment name that the current config has no key for, and how the orch watcher reads the resolved palette color from reed"
---

### [BLOCKING] D7 mis-states how a fixed-height pin behaves under a top border title

**Location:** `_lyx/discussion/decision-record.md:84-90` (D7), and the acceptance criterion at line 253.
**Issue:**
The `pane-border-status top` part of D7 is right.
On tmux 3.6 the window layout still sums to the full height, the top pane sits at `pane_top` 1 with a height one below its cell, and an attached client with `status 2` reads `#{window_height}` as client rows minus 2 whether or not the border status is on.
The told box (`rows - reserved`, `attach.go`) and the live box (`#{window_height}`, `windowsize.go`) are therefore the same box.

The pin sentence is wrong.
D7 says every fixed-height pin "keeps its cell height" and that "where a pin sizes content rather than a cell, the top cell's pin grows by the one title row".
reed's pins are `resize-pane -t <pane> -y <h>` in the `window-resized` hook array (`resizePinHookArgvs`), issued for the Selvage band and for every collapsed placement (`render.FixedHeightPins`).
The top stack strand, the driver, is a collapsed placement whenever the stack has two or more strands, so it is pinned routinely.
Measured on tmux 3.6 with `pane-border-status top`: `resize-pane -t <top> -y 3` yields a pane of 3 rows in a cell of 4 (`80x4` in `#{window_layout}`), taking the row from the neighbouring cell.
So for the top pane `-y` sizes content, never the cell.
A pin that is to keep the planned cell height of 3 must be `-y 2`, a pin that shrinks by one, not "grows".
Followed as written (a pin that grows), the top cell ends one row taller still than the plan.
Left as is, the hook rewrites the planned cell to 4 on every window-resized event, and with the watchdog off the layout stays that way.
The acceptance criterion "fills the window exactly, with no rescale and no overflow, at boot and after a resize" holds for the window sum but does not detect the cell drift, because it never checks per-cell heights.

**Fix:**
State the one intended outcome and its arithmetic.
Either the top cell keeps its planned height and its pin value is the cell height minus the title row, or the top cell is planned one row taller so its content keeps `collapsed_rows`; pick one.
Say that only a pin on the pane at window row 0 differs from its cell, and that a Selvage-only window (`soleBand`, no pin) needs no pin change.
Extend the acceptance criterion to the per-cell heights after a resize with the hook installed.

### [BLOCKING] D13 refuses a bad color at config load, a channel reed's other keys do not use

**Location:** `_lyx/discussion/decision-record.md:160`, and the acceptance criterion at line 257.
**Issue:**
D13 says a value outside the palette "is a config error refused at load ... the same channel as reed's other config keys".
reed's other validated keys (`debug_log`, `mouse`, `watchdog`) are not refused in `LoadConfig`; `LoadConfig` only unmarshals.
They are checked in `ensureServerAndSessionLocked` (`lifecycle.go`) at boot, and elsewhere an invalid `watchdog` value takes a fail-safe side (`windowsize.go`), a deliberate split.
`reedengine.LoadConfig` is also the first step of every `lyx reed` verb (`reedcli/cli.go`, the persistent pre-run), of `pairteardown`'s `reedEngine` (needed for `Down` before a worktree is removed), of the watchdog daemon's `enterSession`, and of the loom and burler wiring.
A color typo refused there would block `lyx reed down`, `status` and `attach` (the escape hatch `AttachArgv` promises never to block), pair teardown and the watchdog for that worktree, all over a display-only value.
The "same channel" premise is false, and the stated refusal has a blast radius the Decision does not bound.

**Fix:**
Validate the segment colors in the boot block next to the `mouse` and `watchdog` checks, naming the key, the value and the palette, and make every other consumer fail safe (an unknown value yields no color, logged) so display never blocks `down`, `attach` or teardown.
Update the Decision, the `PATTERN-config-strictness` constraint line and the acceptance criterion to say "refused at boot".

### [MEDIUM] D12's ordering guarantee has unbounded cases

**Location:** `_lyx/discussion/decision-record.md:131-141` (D12), Open risks line 238, acceptance criterion line 255.
**Issue:**
The rationale says every session is created only after its creator has brought up all its predecessors, so a booter that finds the server already up still fills the earlier gaps first.
That holds for predecessors that are not live.
It does not hold in two cases the Decision's own trigger ("whether or not the server answers") admits:
a booter whose recorded session died while the server stayed up, with later-position sessions still live, creates its session after them, and session ids cannot be reordered without killing live sessions;
and a predecessor whose nested up fails ("logged and skipped") lets the booter create its session before it, so the order is broken, which the acceptance criterion accepts ("a pair whose up fails is skipped and the boot completes").
Open risks names only the server started by hand outside lyx.
The Bound says what the step may bring up but not where the guarantee stops.

**Fix:**
Add both cases to the Bound or to Open risks: order is guaranteed across a full server restart and against not-live predecessors, not against live later sessions or a failed predecessor.
Adjust the acceptance criterion so it does not imply more.

### [MEDIUM] D12 leaves the lock placement and the cross-worktree up mechanism open

**Location:** `_lyx/discussion/decision-record.md:135-137`, Constraints line 204.
**Issue:**
D12 triggers the step "when reed is about to create the booting worktree's session", which today is inside `ensureServerAndSessionLocked`, itself inside `Up`'s `withOpLock`.
The same Decision runs the nested ups "outside the booting worktree's op lock, so no two op locks are ever held at once".
It does not say whether the booter releases its lock, runs the ups and re-takes it (then re-checking that its own session is still absent and its state unchanged), or runs the step before taking it (then reading state unlocked).
A second open point: `reedengine.New` takes one worktree's `Config` and `Geometry`, and reed "derives no path" (`PATTERN-told-geometry`).
D12 says the teller hands over an ordered list with start times, which is not enough to run another worktree's ordinary `up`; the decision does not say what else is told (a per-worktree runner is the obvious shape) or that the nested engine uses that worktree's own `reed.yaml`.

**Fix:**
State the lock sequence (where the trigger is read, where the lock is released and re-taken, what is re-checked), and state what the teller supplies per list entry so that `reedengine` can run an up without deriving a path.

### [MEDIUM] D15's atomic write contradicts its own read-only promise

**Location:** `_lyx/discussion/decision-record.md:188` (and the acceptance criterion at line 259, "an unwritable path").
**Issue:**
D15 writes a temp file beside the target and renames it over, and says "a failed write, a read-only file or directory included, leaves it untouched".
A rename replaces a read-only file when its directory is writable, so for a read-only file in a writable directory the write succeeds and the file is replaced, contrary to the sentence.
The rename also gives the replacement the temp file's mode and owner, not the original's, which D15 leaves unmentioned for a file the operator owns and edits by hand.
A symlinked file is handled ("written at its target"), but a target that is itself in a read-only directory fails at temp creation, which is the only case the sentence covers.

**Fix:**
Decide the intended behavior: either check the file's writability first and skip with a reason when it is read-only, or accept replacing it; and state that the new file keeps the original's mode.
Make the acceptance criterion match.

### [LOW] D15's marker rules leave edge cases open

**Location:** `_lyx/discussion/decision-record.md:182-184` (D15).
**Issue:**
The skip list covers an unmatched begin, a repeated begin and an end before its begin.
It does not say what happens for a begin or end marker outside the top-level array (a replace in place would then put entries outside the array), an end marker with no begin or a repeated end marker, or the line ending of the inserted lines in a CRLF file (a bare `\n` makes a mixed-ending file; `// lyx:end` also needs its own terminating newline so that the first byte after it, on a one-line file such as `[{...}]`, is not swallowed into the comment).
The insertion point itself is sound: inserting after the opening `[` changes no existing byte, a trailing comma before `]` is accepted by VS Code, and only the opening bracket needs to be found, so a scan for the closing bracket is not needed.

**Fix:**
Add one sentence to the skip rule: a marker pair must sit inside the top-level array, exactly one begin and one end are accepted, and the inserted lines use the file's existing line ending and end with a line terminator.

### [LOW] D13 and D14 do not say how an unknown segment or the resolved color is read

**Location:** `_lyx/discussion/decision-record.md:162,170-173`.
**Issue:**
The Decision names "a strand with no segment has no color" but not a persisted strand whose segment string has no key in the current config (a record written by a newer or older lyx).
D14 also says the orch watcher "takes the orch strand's palette color from reed", while the strand record persists only the segment; it does not say which reed surface (its status output) carries the resolved palette color.

**Fix:**
Say that an unknown segment resolves to no color (logged) and name the reed output that returns the resolved color to the orch watcher.

## Focus items settled without a finding

- Told box against live box (focus item 1): the two are the same box; the border title row comes out of the top cell and `#{window_height}` is unchanged by it, checked with an attached pty client on tmux 3.6.
  The defect is the pin arithmetic (F1), not the box.
- Booter's own position and nested-up recursion (focus item 2): the booter's session is created after the earlier list entries and before later ones, and nested ups cannot re-enter the step because each is told to skip it; the open parts are in F3 and F4.
- Palette validation (focus item 4): only palette names reach `@strand_color`, a tmux format or the typed `/color` command, and reed alone reads the config; the open part is the channel (F2) and the unknown-segment case (F7).
- Keybindings insertion (focus item 5): inserting after the opening `[` with comma-terminated entries leaves every existing byte unchanged and needs only the opening bracket, found by the comment- and string-skipping scan; the unmatched-begin skip is stated; the open parts are F5 and F6.
