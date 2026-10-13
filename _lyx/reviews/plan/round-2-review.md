---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/05-window-order-and-shell.md:30
    summary: OpenWindow routes its new-window through newWindowLocked in spawn.go, so window.go loses its only new-window literal and windowCreationAllowlist's window.go entry goes stale, failing TestPaneCreationSitesRouteThroughThePreludeChokepoint, and card 5 does not target the allowlist
    fix: In card 5, add `internal/reedengine#windowCreationAllowlist` to the `**Edit:**` list (after `internal/reedengine#TestAttachArgv_EveryOtherDegradedPathYieldsBareArgv`), and add an Intent sentence after the one naming `newWindowLocked`'s file - `windowCreationAllowlist` in `panebin_enforcement_test.go` drops its `internal/reedengine/window.go` entry, whose `new-window` literal moves into `newWindowLocked`, and its `spawn.go` entry's `Why` names the one window-creation helper that both own windows and `OpenWindow` open through, since `RequireNoStale` fails on an entry that matches no file. Checked with `lyx loom validate-plan --overlay`, which reports no new finding.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-focus-rules.md:47
    summary: Four internal/reedengine/doc.go sentences that no card names for a rewrite contradict the code cards 5, 7 and 8 add - the OpenWindow argv with -t on the session, own windows that never take focus, a removal that moves the viewer to the strands' window, and the VIEW button in the geometry-pins list
    fix: Name each sentence in the card whose code falsifies it. Card 5, beside its `@lyx_hidden` rewrite - the Detached windows sentence giving `new-window -d -P -F '#{window_id}' -t '=<session>:' -n <temporary> <command>` is rewritten with the `-a -t <highest-index window>` placement and its plain-form fallback. Card 7, in its doc sentence - the Own windows sentence ending `by new-window -d, so it never takes focus` is rewritten to say a new agent window is selected under the focus rule, and the sentence `Removing an own-window strand moves a viewer on its window to the strands' window before the window ends` is rewritten to say the viewer goes to the relevant window, never the shell. Card 8, in its doc sentence - the geometry-pins bullet's `"status-format[0]" (the VIEW, strand and window buttons)` becomes the strand, window and `shell` buttons.
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-window-order-and-shell.md:61
    summary: TestWindowOrder's pure rows for a listed shell, a listed old window and the overview's lowest-window target pin the same placement TestWindowOrder_RealTmux's rows pin at the tmux tier, two tests for one behavior per PATTERN-test-economy
    fix: In card 5, drop TestWindowOrder's rows "a listed old window gives `-a -t <old window>` even with a shell listed" and "a listed shell gives `-b -t <shell>`", which TestWindowOrder_RealTmux's rows "three own-window strands added in turn hold ascending indexes before the shell" and "relaunching a dead agent through `AddStrand` with `IfAbsent` keeps its window between its neighbours" cover, and keep its no-shell and no-window rows, which no tmux row reaches. In card 6, drop the `overviewWindowInsertArgs` row "the lowest window is the target", which the TestWindowOrder_RealTmux row "an overview strand added after two agent windows holds the lowest index" covers, and keep the hidden-lowest-window and no-window rows. Change the sentence stating what TestWindowOrder covers to say it holds only the fallback cases the real-tmux rows cannot reach.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-window-order-and-shell.md:45
    summary: Card 5 leaves two parts of the OpenWindow change unstated - what a missing reed.json (LoadState returning nil, nil, which every TestOpenWindow row's newTestEngine has) means for ensureShellWindowLocked and shellWindowOf, and that TestOpenWindow's exact wantArgv must be rebuilt around the placement slot newWindowLocked inserts
    fix: In card 5's `OpenWindow` sentence, add that a nil state (no reed.json) records no Selvage pane, so the shell check does nothing and logs at Debug. In the `TestOpenWindow` sentence, add that its exact `wantArgv` is rebuilt as `new-window -d` plus the placement (`-t` on the exact session target when `list-windows` is unscripted) plus the caller's arguments, for every existing row, beside the change that finds the temporary name after `-n`.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/06-overview-anchor.md:52
    summary: The overview window-group refusal fires on Display.Window non-empty or WindowSize non-zero, but the validateDisplay subtest only names an overview display carrying Window, so no test reaches the WindowSize-only branch
    fix: In card 6's `validateDisplay` subtest sentence, add a third case - an overview display with `Window` empty and `WindowSize` non-zero wants the window-group refusal.
---

### [BLOCKING] Moving `OpenWindow`'s `new-window` into spawn.go leaves a stale allowlist entry

**Location:** `_lyx/plan/05-window-order-and-shell.md:30`

**Issue:** Card 5 has `newWindowLocked` apply every placement, the psmux retry included, and has `OpenWindow` open its hidden window with `hiddenWindowInsertArgs`.
Both callers then go through `newWindowLocked`, the reading focus entry 1 confirms.
`newWindowLocked` lives in `spawn.go`, and the card justifies that by `windowCreationAllowlist` admitting the literal "only there and in window.go".
Once `OpenWindow` calls it, `internal/reedengine/window.go:82` holds the file's only `"new-window"` literal, and that literal leaves; the doc comment at `window.go:49` is no literal.
`TestPaneCreationSitesRouteThroughThePreludeChokepoint` (`panebin_enforcement_test.go:66`) calls `allow.RequireNoStale(t)`, and `scankit`'s `Allowed` marks an entry hit only for a file that holds the literal (`internal/testkit/scankit/scankit.go:227`).
So the `window.go` entry turns stale and `go test ./internal/reedengine` fails.
No card targets `windowCreationAllowlist`, so a scope-guarded implementer cannot repair it in card 5.
The `spawn.go` entry's `Why` ("the launch chokepoint's own-window branch") also stops describing what the literal serves.

### [MEDIUM] doc.go sentences no card rewrites contradict the new code

**Location:** `_lyx/plan/07-focus-rules.md:47`

**Issue:** The Constraints require `internal/reedengine/doc.go` to land with the code.
Card 5 names only one sentence for a rewrite (the `@lyx_hidden` line), and cards 7 and 8 refer to "the focus section" and "the bindings and bar paragraphs", which `doc.go` does not have as such today.
These sentences stay, and each is falsified:
`doc.go:306` gives OpenWindow's argv as `-t '=<session>:'`, but card 5 opens it `-a -t <highest-index window>`;
`doc.go:317` says an own window "never takes focus", but card 7 selects a new agent window from the overview or a dead agent window, and for `Focus: true`;
`doc.go:328` says removal moves the viewer "to the strands' window", which is the shell, against D5's "never the shell";
`doc.go:862` lists "the VIEW, strand and window buttons" in the geometry-pins bullet, outside the bindings paragraph card 8 names.
Focus entry 5 asks about this case; the `@lyx_hidden` sentence itself is handled by card 5's rewrite.

### [MEDIUM] Pure placement rows duplicate the real-tmux order rows

**Location:** `_lyx/plan/05-window-order-and-shell.md:61`

**Issue:** `PATTERN-test-economy` tests an unexported helper through the surface that uses it, and treats a redundant test as a defect.
Card 5's `TestWindowOrder` rows for the listed shell (`-b -t <shell>`) and the listed old window (`-a -t <old window>`) pin the same placement as `TestWindowOrder_RealTmux`'s ascending-indexes-before-the-shell row and its relaunch-keeps-its-window row.
Card 6 repeats this: its `overviewWindowInsertArgs` row "the lowest window is the target" pins what the real-tmux row "an overview strand added after two agent windows holds the lowest index" pins.
The fallback rows (no shell, no window, a hidden lowest window) are reached by no tmux row and are kept.
Focus entry 4 asks about this case.
For card 7 the answer is no: it pins `relevantWindow` and `newAgentWindowSelected` through their callers only.
Card 5's `TestOpenWindow` hidden-window argv row is not a duplicate of the real-tmux hidden row: it pins exactly one `-t`, which a real tmux run cannot see.

### [LOW] `OpenWindow`'s nil state and `TestOpenWindow`'s exact argv go unstated

**Location:** `_lyx/plan/05-window-order-and-shell.md:45`

**Issue:** `LoadState` returns `nil, nil` when no `reed.json` exists (`internal/reedengine/state.go:158`), and `newTestEngine` writes none, so every `TestOpenWindow` row reaches `ensureShellWindowLocked` with a nil state.
The card covers only a load failure.
`shellWindowOf(st, …)` reading `st.SelvagePaneID` would panic on nil unless the card says nil means no recorded Selvage pane.
`TestOpenWindow` also compares the whole `new-window` argv exactly (`window_test.go`'s `wantArgv`, `-d -P -F … -t <session> -n …`).
`newWindowLocked`'s "`new-window -d` plus the placement plus the caller's arguments" moves `-t` ahead of `-P`.
The card mentions only the `-n` index change, so it leaves out that every row's `wantArgv` must be rebuilt.

### [LOW] The `WindowSize`-only branch of the overview window-group refusal has no test

**Location:** `_lyx/plan/06-overview-anchor.md:52`

**Issue:** Card 6 refuses an overview display when `Display.Window` is non-empty or `WindowSize` is non-zero.
The `render.ValidateDisplay` it calls first accepts an overview display whatever its size, so both inputs reach the refusal.
The named subtest only passes an overview display that carries `Window`, so removing the `WindowSize` clause fails no test.

## Rejected

- Focus entry 1: every `new-window` argv carries exactly one `-t`, and for a placed window it names the placement window, not the session. `ownWindowInsertArgs`, `hiddenWindowInsertArgs` and card 6's `overviewWindowInsertArgs` each return `-a`/`-b -t <window id>`. `newWindowLocked` adds the session `-t` only for an empty placement, which includes the psmux retry. This holds for both callers. No defect beyond F1 and F4.
- Focus entry 2: card 6 states its bound in `**Intent:**` ("one strand per session, alone in a window of its own that opens at the lowest index, and it never joins a group"). A grouped overview display is refused in `validateDisplay` before `planOwnWindowPlacement` can join a window. The second-overview refusal is reached by a dormant overview strand under another name, past the name check, and fails on a missing guard because its assertion names the holder and `lyx reed remove`. The window-move refusals are reached by the two `TestUpdateStrandLocked` rows. The only gap is F5.
- Focus entry 3: card 7's deciding guards each have a row whose fixture makes the guard decide. These are another client attached, the failed client listing (`ListClientsError`, seeded with a live own-window strand so a missing guard appends a selection), the new-window selection matrix and the end-of-window selection. Its remaining degrade branches are a failed current-window read, a failed window listing and "none left". Each yields the same outcome as an absent guard (no selection), so no fixture can make one decide, and no finding is raised for them.
- Card 8's removal of the `@lyx_strands` gate on Alt+Up and Alt+Down lets no key through that was passed before: the old `if-shell` had no else branch, so outside the strands' window the key did nothing (`bindings.go:71`).
- The `overview` anchor stays out of `lyx reed add --anchor`, whose closed vocabulary `internal/reedcli/add.go:75` keeps. D1 asks for no CLI exposure, so no card is missing there.
