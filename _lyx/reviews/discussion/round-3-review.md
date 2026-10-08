---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D7 adjusts the pin of the pane at window row 0 but never says which order decides row 0, and the pin function it builds on reports no position
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D7 subtracts the title row from the top pin unconditionally although D6 makes the pane-border-status pin non-fatal, so a refused pin leaves the top cell one row short on every resize
  - id: F3
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:89
    summary: D7 states no floor for the top pin when the planned top cell is one row, where the pin would be -y 0 and the claim that every cell keeps its planned height cannot hold
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:138
    summary: D12 has reedengine decide whether a predecessor's reed state records a session that is not live, but the teller supplies only name, spawn-order key and an up function, so the state location and session name that decision needs are not told
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:247
    summary: "D12's stop is stated in three places that do not agree: the Bound names the live later-position stop, the Open risks bullet only points at it, and no acceptance criterion pins it; the bullet is also fused to the next risk on one line"
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:100
    summary: D8 puts tmux runtime format values inside single quotes in a run-shell command string, and neither D8 nor D11 says what bounds a socket path or client name containing a quote
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:197
    summary: "D15's atomic write, its skip list, its Bound and its acceptance criterion differ on three points: a failed write is not a listed skip, the read-only check and the identical-content no-op have no order, and the Bound omits the temporary file"
---

### [MEDIUM] D7's "pane at window row 0" has no defined source

**Location:** `_lyx/discussion/decision-record.md:89`, acceptance criterion at line 261.
**Issue:** D7 says the pin on "the pane at window row 0" is its planned cell height minus one, and every other pin keeps its cell height.
The pins come from `render.FixedHeightPins`, which `internal/reedengine/render/rules.go` documents as taking no `paneOrder`: it emits pins in the intended parent-above-child order, by pane id, with no position.
`Rules` does take `paneOrder` and puts the physically first pane at row 0, keeping each pane's height with its id (`resequenceByPaneOrder`).
After any swap or reorder, the pane physically at window row 0 is not the first placement.
It can also be the bottom-most strand, which has no pin at all.
The record says neither which pane D7 means nor where reed learns it.
A plan writer who takes "row 0" from `FixedHeightPins`' first strip pin adjusts the wrong pane after a reorder.
The one-row error then lands on a pane that has no title row and leaves the real top cell one row too tall.
**Fix:** In D7, state that row 0 is the first cell of the layout actually applied, in the physical order that `select-layout` string uses, and that the adjustment is made on the pins computed from that same order.
Add to the acceptance criterion a case with the strand panes in a non-intended physical order.

### [MEDIUM] D7's top-pin adjustment ignores a refused `pane-border-status`

**Location:** `_lyx/discussion/decision-record.md:89`, with D6 at line 80.
**Issue:** D6 makes every new pin non-fatal: a failed `pane-border-status top` set is logged and the session keeps working.
D7 subtracts the title row from the top pin unconditionally.
When the border-status set was refused (an older or partial tmux, or a failed round trip), no title row exists.
The `-y` pin then sizes the top cell one row short on every `window-resized`, and the freed row goes to the bottom pane.
This is exactly the per-resize drift D7's rationale says the adjustment prevents, reached by the degrade path D6 allows.
D7 already takes the layout's reserved rows from a `#{status}` readback rather than from what reed intended to set, so the same rule is missing for the border row.
**Fix:** In D7, key the adjustment on the window's effective `pane-border-status` read back from tmux, as the reserved rows are keyed on `#{status}`; with a readback of `off`, or a failed readback, the pin keeps its cell height.
Add the refused-pin case to the acceptance criterion.

### [LOW] D7 has no floor for a one-row top cell

**Location:** `_lyx/discussion/decision-record.md:89`.
**Issue:** The height planner floors every cell at one row (`clampToFit`), so a short window can plan a one-row collapsed cell at the top.
Its adjusted pin would be `resize-pane -y 0`, and a pane with the title row has no content row left.
D7's statement that "every planned cell keeps its planned height" does not hold there, and the record names no bound.
**Fix:** State in D7 what happens when the planned top cell is one row: the pin is floored at one content row and the cell is allowed to be one row taller, or the pin is skipped.
Either is acceptable; the record must pick one.

### [MEDIUM] D12's teller input does not cover what reedengine is asked to decide

**Location:** `_lyx/discussion/decision-record.md:138`, `:140`, `:142`.
**Issue:** The trigger and the Bound make reedengine bring up "every earlier worktree whose reed state records a session that is not live".
Deciding that needs each predecessor's reed state directory and the name of its recorded session, to read the state and probe the socket.
The teller supplies per entry only the worktree's name, its spawn-order key and an up function.
`PATTERN-told-geometry` forbids reedengine from deriving the state path, and the same sentence says reedengine "reads no loom or fabric record itself".
Two readings fit the text.
Under one, the teller also supplies the state location and session name, and reedengine does the liveness test.
Under the other, the up function carries the precondition and answers "nothing to do" for a worktree with no recorded session or a live one.
They put the "recorded and not live" test in different packages, with different tests.
**Fix:** In D12, state which side decides, and list what the teller supplies per entry to match: either add the predecessor's state location and session name to the entry, or say the up function reports "skipped" for a worktree that has no recorded session or whose session is live, and reedengine only sequences.

### [LOW] D12's ordering stop is not stated the same way in the Bound, the Open risks and the acceptance criteria

**Location:** `_lyx/discussion/decision-record.md:143`, `:247`, `:263`.
**Issue:** The Bound states two stops: a later-position session that is still live, and a predecessor whose up failed.
The Open risks bullet restates only the hand-started server and points at "the two revivals" the Bound names, without saying what they are.
The acceptance criterion pins the failed-predecessor stop and the first-boot and downed-worktree exclusions, and has no case for the live later-position stop.
A revival on a server that stayed up, which creates the booter's session after live later sessions without hanging or reordering them, is therefore a stated behavior with no scenario.
Line 247 also fuses two risks into one line (`...out of order.- A strand recorded before this change...`), so they render as one bullet and break the one-sentence-per-line rule.
**Fix:** Add one acceptance case for the live later-position stop.
Name the two stops in the Open risks bullet, or drop its pointer to the Bound.
Split line 247 into two bullets.

### [LOW] D8's runtime-expanded values are single-quoted in a shell string with no stated bound

**Location:** `_lyx/discussion/decision-record.md:100`, with D11 at line 129.
**Issue:** The `M-Left`/`M-Right` binding runs `lyx reed switch --socket '#{socket_path}' --client '#{client_name}' --tmux <path>`.
tmux expands the formats when the key fires, after the string was built, so `internal/shell` cannot quote them.
A socket path or client name containing a single quote ends the quoted token and lets the rest be parsed by the shell.
D11 bounds the `lyx` token only.
D10's Bound limits what the verb runs, not what the binding's string can carry into the shell.
The socket path and tmux path are known when the binding is pinned; only the client is known at press time.
**Fix:** In D8, either pin the socket and tmux paths as shell-quoted tokens at bind time and leave only `#{client_name}` to tmux, or state that a path containing a single quote is refused when binding, and what tmux's `client_name` can contain.

### [LOW] D15's write-safety statements differ on three points

**Location:** `_lyx/discussion/decision-record.md:196`, `:197`, `:200`, acceptance at line 267.
**Issue:** The record gives the skip conditions, the atomic write, the Bound and the acceptance criterion, and they differ in three places.
First, D15 lists the read-only file as a skip, but a failed write or unwritable directory only "leaves it untouched", and the criterion counts an unwritable directory as a reported skip; D15 does not say it is reported as one.
Second, D15 skips a read-only file "before anything is written" and also says a content-identical result "writes nothing".
For a read-only file that already holds the block, the order decides whether the run reports a skip or a no-op.
Third, the Bound says the verb writes only "in that one file", but the write goes through a temporary file in the same directory, and its cleanup on a failed rename is not stated.
**Fix:** In D15, state that a failed write, directory included, is reported as a skip with its reason, put the identical-content check before the read-only check (or the reverse, and say so), and widen the Bound to the temporary file, removed on every path.

## Focus verdicts

- **Pin arithmetic (D7):** the value is right for a top pane that is a collapsed strip, and the criterion checks per-cell heights as the focus asks.
  The unresolved points are F1, F2 and F3.
- **Color validation channel (D13):** refusal in `ensureServerAndSessionLocked`'s pre-tmux block is the same point as `mouse` and `watchdog`.
  `down`, `status`, `pairteardown`, the watchdog and an attach to a live session never reach it (`ensureSessionLocked` returns early on a usable session).
  Not a defect.
  The same block does run for `Resume` and for the cold-boot self-heal of a strand add, so a typo also stops those; this matches how `mouse` already behaves, and I did not raise it.
- **Ordering bound (D12):** F5.
- **Lock sequence and teller input (D12):** the lock release, re-take and re-check are stated.
  The teller input is F4.
- **Atomic write (D15):** F7; the mode kept on a replaced file is stated consistently in the decision and the criterion.
