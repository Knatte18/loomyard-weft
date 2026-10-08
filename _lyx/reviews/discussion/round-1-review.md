---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:86
    summary: D7 rests on a false premise, since pane-border-status top takes its row inside the top pane's cell and the layout must still sum to the window height, so subtracting a row leaves the layout one row short
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:131
    summary: D12 creates the booter's own session last and leaves the nested ups' own boot step unbounded, which contradicts the prime-first spawn order it exists to keep
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:136
    summary: D12 rationale that concurrent booters each create a session only after its predecessor exists fails for a booter that finds the server already up, because the step runs only when no server answers
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:172
    summary: D15 inserts the block before the closing bracket but leaves open the comma the previous last entry then needs, how the bracket is found in a JSONC file, and a begin marker with no end marker, contradicting the byte-identical criterion
  - id: F5
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:187
    summary: The config-sourced colors decision is appended without an id and D13, D14, Scope, the constraints and the acceptance criteria still describe hard-coded colors
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:189
    summary: Config-supplied color values flow into a tmux option and a command typed into an agent pane with no stated validation bound, and the decision does not say which module reads the config for shuttle and the orch watcher
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:140
    summary: D13 puts Claude Code palette color names in a shared leaf package while the constraints say the Claude color names live only under claudeengine
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:239
    summary: The acceptance criterion requires @strand_color on a pane spawned before this change, but D2 and D14 derive it from a segment persisted at add time that an older strand record does not have
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md:62
    summary: The support log records no round or ledger entry for the parent change that added the config-colors decision, and the record still says no operator answered
---

### [BLOCKING] D7 subtracts a row the layout never loses

**Location:** `_lyx/discussion/decision-record.md:86` (D7), with `:243` (acceptance criterion on the exact fill).
**Issue:** D7 says the row budget "subtracts the one row `pane-border-status top` adds at the top of the strand window", on the premise that an unaccounted row makes tmux rescale.
I checked this on tmux 3.6 (a 30-row window, three stacked panes, `pane-border-status top`).
`#{window_layout}` is `100x30,0,0[100x15,…,100x7,…,100x6,…]`, which sums to the full window height with and without the option.
The border row is carved out of the top cell: `pane_top` is 1 and `pane_height` is 14 for a cell of 15, and the same holds with the pane zoomed.
Selecting a layout one row short is not accepted as an exact fill: tmux rewrote the window layout to `100x29`.
The live-window path takes its box from `#{window_height}`, which already excludes the status rows, and nothing in D7 subtracts a row from it.
Following D7 as written would make the told-box path in the attach pre-flight one row shorter than the live path, so the acceptance criterion "fills the window exactly, with no rescale and no overflow, at boot and after a resize" fails.
The visible effect of the option is one less content row in the top (driver) pane, not a smaller budget.
**Fix:** Replace the subtraction with the actual rule: the status rows come off the budget as today (`2` reserves 2), the border-status row is inside the top cell and costs the layout nothing, and the driver's content height is one row less.
State that the layout sums to the window height.
Check that the fixed-height pins (driver, collapsed cells, Selvage band) still hold when the top cell's content shrinks by one.

### [BLOCKING] D12 puts the booter's own session last and leaves its recursion unbounded

**Location:** `_lyx/discussion/decision-record.md:131-136` (D12), `:245` (acceptance criterion).
**Issue:** The Goal and the criterion want the prime first and each pair in start order.
D12 has the booting worktree bring up "the session of every hub worktree that had one" and "only then" its own, and the criterion says "before its own".
A booter that is the prime therefore creates its session after the pairs, and a booter that is the second pair creates its session after the third pair's.
The order D5 and D10 rely on holds only for a booter that happens to be last in spawn order.
Each nested up is "that worktree's ordinary `up`", and an ordinary up on a hub with no server is itself the boot that D12 triggers.
Nothing says the nested ups skip the step, so as written the first nested up re-enters the pre-boot, and the Bound ("runs only when no server answers") is true for it too.
**Fix:** State that the booter's own session is created at its own position in the ordered list (the list includes the booter, and the booter's turn comes after its predecessors and before its successors), and that the nested ups are told not to run the pre-boot step.
Correct the criterion to say the sessions appear in spawn order, the booter's included.

### [MEDIUM] D12's concurrency claim fails for a booter that finds the server up

**Location:** `_lyx/discussion/decision-record.md:136` (rationale), `:135` (Bound).
**Issue:** The rationale says concurrent booters iterate the same list, so each session is created only after its predecessor exists.
That holds only while both booters saw no server.
Once the first booter's first up has created the server, a second booter arriving a moment later finds it up, skips the step by the Bound, and creates its own session at once, possibly before the first booter has reached that worktree's predecessors.
Booters do not wait on each other (no op lock is held across the step), so there is no deadlock, but there is a creation-order race.
Batten revival after a restart, where several pairs boot close together, is the realistic trigger, and the Open risks name only a hand-started server.
**Fix:** Either state the guarantee as best-effort for concurrent booters and name this case in Open risks, or bound the step on a condition the late booter can also see (for example, a session missing from the ordered list while the list's earlier members are absent) so a late booter fills the earlier gaps first.
Do not claim the order for concurrent booters otherwise.

### [MEDIUM] D15 cannot meet its own byte-identical bound

**Location:** `_lyx/discussion/decision-record.md:172-176` (D15), `:248` (criterion).
**Issue:** D15 inserts the block "before its closing `]`" and promises that nothing outside the markers is changed.
When the file's last entry has no trailing comma, a block placed before `]` needs a comma added after that entry, which is a byte outside the markers; the criterion "leaves everything outside the markers byte-identical" cannot hold.
VS Code accepts trailing commas in this file, so inserting right after the opening `[` (block entries each ending in a comma) needs no outside edit, but D15 does not choose.
The decision also gives no rule for finding the closing bracket in a JSONC file with `//` comments or `]` inside strings (`"when": "a[0]"`), no rule for a `// lyx:begin` with no `// lyx:end` (a replace in place would otherwise run to end of file), and no rule for a keybindings file that is a symlink or a read-only path.
This verb writes a file outside the repo, so the bound has to hold for every file it accepts.
**Fix:** Choose the insertion point so no outside byte changes, state how the array bounds are found in JSONC (comments and strings skipped), skip with a reported reason when a begin marker has no end marker, and state that the write replaces the file atomically or leaves it untouched.

### [MEDIUM] The config-colors decision is bolted on and leaves the record inconsistent

**Location:** `_lyx/discussion/decision-record.md:187-191`, against `:18`, `:138-155`, `:161`, `:246`.
**Issue:** The added decision has no `D<n>` id and carries its provenance in the heading ("Added after Discussion (parent, 2026-10-08)"), which is history, not a decision.
It says nothing hard-codes a color in Go, but D13 still "declares one constant per loom segment" mapped to colors, and D14 sets `@strand_color` from the persisted segment alone.
Scope lists no configurable colors, no constraint covers the new reed config keys (their template default, a missing key in an existing `reed.yaml`), and the criterion "`@strand_color` matches D13's table" holds only with default config.
The Auto-mode assumptions still call the table "picked by this agent" without saying the operator now owns the values.
**Fix:** Fold the decision into D13 (and D14 where the color is read), so D13 states the segments as constants, the palette defaults as the config template and the config as the source of the live values.
Add the config keys to Scope and Constraints, and make the criterion say the color matches the configured value, the default being D13's table.

### [MEDIUM] Config color values have no stated bound, and the reader is unnamed

**Location:** `_lyx/discussion/decision-record.md:189` (added decision), `:161-163` (D14).
**Issue:** A value read from config now reaches two sinks: a tmux option (`@strand_color`, interpolated into status and border formats) and the text `/color <name>` typed into an agent pane (and the orch pane).
The attack-surface question applies: the Decision does not say that the value must be one of the palette colors, so a typo or a value carrying format syntax (`#[`, `}`) or whitespace could break the bar for every session or type arbitrary text into an agent's input.
The Decision also does not say who resolves a segment to its configured color: the colors are reed's config, but shuttle (which types the command) and the orch watcher (which types the orch's) are other modules, and `reedengine` is told, not given, other modules' state.
**Fix:** State that each configured value is validated against the closed palette at load, naming the outcome for an unknown value (refuse at boot with a way forward, or fall back to the default and log), and state which layer reads the config and tells the color to reed, shuttle and the orch watcher.

### [MEDIUM] D13 and the provider-seam constraint disagree on where Claude color names live

**Location:** `_lyx/discussion/decision-record.md:140-150` (D13), `:201` (constraint).
**Issue:** D13 calls the table's color column provider-neutral but defines it as "Claude Code's `/color` palette" (blue, purple, orange, pink, …), in a leaf package outside `claudeengine`.
The constraint says `/color` and the Claude color names live only under `claudeengine`, and `PATTERN-shuttle-provider-seam` bars Claude specifics from `shuttleengine` and `reedengine`.
Both cannot hold: either the shared package carries Claude's names, or `claudeengine` owns the segment-to-name map.
**Fix:** Pick one.
Either the shared package maps segment to tmux color only and `claudeengine` holds the segment-to-Claude-name map, or the shared package keeps the palette and the constraint and `PATTERN-shuttle-provider-seam` entry are widened in the same commit.

### [MEDIUM] The pre-change-pane criterion outruns D2 and D14

**Location:** `_lyx/discussion/decision-record.md:239` (criterion), `:38-39` (D2), `:161` (D14).
**Issue:** The criterion requires `@strand` and `@strand_color` on "a pane spawned before this change" after an attach.
`@strand` is derived from the strand's name and works.
`@strand_color` comes from a segment that the spawning module names at add time and reed persists on the record; a record written before this change has no segment, and reed has no name-to-segment map (the spawning module owns it).
So that pane gets no color, which D2 allows ("unset when the strand has none") but the criterion forbids.
The Open risks entry on `--resume` and `/color` covers Claude Code's prompt bar, not this tint.
**Fix:** Limit the criterion to `@strand` for pre-change panes and to both options for strands recorded by the new lyx, and say that an older strand has no tint until it is respawned.

### [LOW] Support log does not record the parent change

**Location:** `_lyx/discussion/support-log.md:60-62` and `:64-82`, against `_lyx/discussion/decision-record.md:187-191` and `:216`.
**Issue:** The added decision is an operator-sourced change of 2026-10-08, yet `## Review rounds` still reads `_No rounds yet._`, the Question ledger has no entry for it (Q13 settles "per segment" only), and the Auto-mode assumptions still say no operator answered and every decision was picked by this agent.
**Fix:** Append the parent-review entry and a ledger entry for the config-colors question, and correct the auto-mode line to say which decision came from the operator.
