# Decision record — reed-add-parentless-placement

## Goal

Every strand reed places gets a tmux window of its own (or its group's), and the selvage `shell` window holds only the Selvage terminal.
`lyx reed add` with no `--anchor` lands a strand in its own window, and the pane-era placement that split strands into the shell window under a parent is retired from reed's vocabulary, layout code, config, bindings, help texts, docs and tests.
The CLAUDE.md workaround for #673 is removed in the same change.

## Scope

In: the anchor vocabulary (D1), the two CLIs' `--anchor` (D2), the engine's display check and the upgrade of old records and panes (D3), the shell window's stacked layout (D4), the layout-only config keys (D5), the resize self-heal (D6), the shell-window pane buttons and Alt+z (D7), and docs, sandbox suites, tests and CLAUDE.md (D8).
The stencils under `contracts/stencils` and `plugins/` were swept and carry no pane-era placement wording, so none changes.

Out:

- Own-window placement itself: window groups, `window_share_max`, `window_split`, window order, focus and follow rules stay as they are.
- `--parent`: it stays with its non-layout meaning (lineage, recursive remove), and its flag help is already accurate.
- The per-hub watchdog daemon: it stays as a per-hub process with its name repair; note `spawn-audit-log` plans a process sampler on it.
- Selvage itself: it stays the always-on operator console in the shell window.
- The status bar's window buttons, the shell button, line 2 and the follow button, which the notes `reed-status-buttons` and `reed-status-buttons-prototype` build on.

## Decisions

### D1 — Retire `below-parent` and `top` from the anchor vocabulary

Decision: `render.Anchor`'s closed set becomes `own-window`, `overview` and `hidden`.
No strand reed places lands in the shell window.
Rationale: the operator's decision names `below-parent` a leftover of the pane era, and the shell window must hold only the Selvage terminal.
No production spawner names `below-parent` or `top`; only `lyx reed add`'s default produces `below-parent`.
Changing the default alone would leave a flag value that still puts a strand in the shell window.

### D2 — `lyx reed add` and `lyx shuttle run` default to `own-window`

Decision: both CLIs default `--anchor` to `own-window` and accept `own-window|hidden`.
Any other value, a retired one included, is refused with a message naming the value and `own-window` as the way forward, and adds nothing; the flag help lists only the accepted values.
`reed add`'s `Long` text says a strand gets its own window, not "a live pane".
Rationale: the CLIs match the window-per-strand layout, and a refusal is clearer than a silent remap for a caller who typed the retired value.

### D3 — The engine refuses a retired anchor on write and migrates old state

Decision: the display check every add (`AddStrand`, `AddStrandUnless`), `UpdateStrand` and `ReplaceStrand` run refuses a display naming `below-parent` or `top`, naming `own-window`.
A `reed.json` strand record carrying a retired anchor loads (PATTERN-tolerant-records), and its anchor reads as `own-window`.
The `size` and `contentRows` keys become unknown keys: no code reads them, and `internal/state` keeps them on a rewrite.
At the first op under the op lock that reconciles the session (up, resume, attach pre-flight, any add or remove), a strand whose pane still sits in the shell window is moved into its own agent window with `break-pane`, keeping its process, then marked and placed as a newly opened own window (before the shell window, carrying `@lyx_own` and the strand's pane options), and its record is rewritten with `anchor: own-window`.
A dead pane is moved the same way, so it relaunches like any dead own-window pane.
Rationale: a running hub may hold a strand an older binary placed; moving it once keeps the operator's live session and lets every own-window rule (relaunch kill, window order, remove, viewer moves) apply unchanged, where a strand left in the shell window would need a special case in each.
Bound: the move touches only bound strand panes found in the shell window, never Selvage and never an unbound pane, which the existing untracked reap still governs.

### D4 — Remove the stacked strand layout of the shell window

Decision: render's stack policy goes: the top and below-parent partition, the stack ordering by insertion, the positional, content and sharing height rules, the band clamp, the focus pick within the window, and the window-layout string reed computed for the shell window.
`Display.Size`, `SetContentRows` and the strand's `ContentRows` go with it, since only the stack read them and no production caller reports content rows.
Reed's split-target planning for a strand pane in the shell window goes, and reed issues no `select-layout` on the shell window.
The shell window's `pane-border-status`/`pane-border-format` pin goes, since only stacked strand panes showed a title there.
Selvage is the shell window's only pane and fills it after a fresh boot, after a Selvage heal (also when the Selvage corpse is the window's only pane, which must not end the window), and after D3's move.
`render` keeps the closed display vocabulary (`Anchor`, `Display`) that spawners name.
Rationale: with Selvage alone in the shell window there is nothing to stack, and code no strand can reach is the pane-era leftover the sweep removes.

### D5 — Retire the layout-only config keys

Decision: `collapsed_rows`, `min_full_rows` and the `selvage` block with its `height_rows` become `RetiredKeys` of reed's config spec with `CategoryNone`, leave both templates, and are dropped on load, so a hub whose `reed.yaml` still holds them loads and `lyx config reconcile` prunes them.
`width` and `height` stay, since the unattached session size still reads them; their comments drop the stack wording.
Rationale: configengine's retired-key mechanism carries this case, as `collapsed_strip_rows` did.

### D6 — Retire the shell-window resize self-heal

Decision: the per-worktree watch loop's layout re-apply (`reapplyLayout` and the `Engine.Watch` loop driving it), the session's `window-resized` hook and its resize signal file go.
Boot and every attach pre-flight clear the `window-resized` hook on the shell window with the existing unconditional unpin (`set-hook -u`), so an entry any older or `.dev-bin` binary pinned is cleared whatever its command string.
The per-hub watchdog daemon keeps its per-session watch, which is now the name repair loop alone.
The `watchdog` key stays and gates that watch, as its "off" spelling already does in the daemon; its template comment is rewritten to name name repair and drop the resize wording.
Rationale: the self-heal exists to re-render the stacked shell-window layout after a resize, and a window holding Selvage alone follows a resize without help.
Bound: the unpin clears only the shell window's `window-resized` hook, which only reed pinned.

### D7 — Retire the shell-window pane buttons, the strand click and Alt+z

Decision: line 1 of the status bar keeps the overview, agent-window and shell buttons, and drops the per-strand pane buttons that followed the shell button.
The strand-button click branch and the Alt+z zoom binding go, and so does the zoom-keeping wrapper that guarded ops against a zoomed strand pane.
The shell click selects the shell window and no longer unzooms.
The bindings pin issues an explicit `unbind-key` for `M-z`, since a `bind-key` re-pin never removes a binding; the click binding is re-bound whole, so its retired branch goes with the re-pin.
Rationale: each of these addresses a strand pane inside the shell window, which D1 makes impossible.

### D8 — Sweep docs, help, sandbox and tests in the same change

Decision: the package docs (`internal/reedengine/doc.go`, the `internal/reedengine/render` package doc, `internal/reedcli/doc.go`), `docs/overview.md`'s reed module-table row ("arranges them", own-window as the exception, the watchdog as a resize daemon) and its spawn walk-through (step 4 names `anchor:below-parent`), the reed config template comments, and `internal/shuttleengine`'s `Display` comment lose every claim that a strand is split under a parent or stacked in the shell window.
Both reed sandbox suites, `tools/sandbox/SANDBOX-REED-SUITE.md` and `tools/sandbox/SANDBOX-REED-WATCH-SUITE.md`, are swept: a scenario that exercises only the retired layout or the resize self-heal is retired with a one-line reason, as M6 already is, and a scenario that adds a strand for another purpose adds it with the default anchor and watches its own window.
Tests that pin retired behaviour are deleted; tests that used `below-parent` only as a convenient anchor move to `own-window`.
The CLAUDE.md #673 workaround is removed, and the `## Workarounds` section with it when left empty.
Rationale: CLAUDE.md's "Docs land in the same commit" rule and PATTERN-sandbox-coverage.

## Constraints

- PATTERN-tolerant-records: an old `reed.json` with a retired anchor, `size` or `contentRows` key loads, and a rewrite through `internal/state` keeps the unknown keys.
- PATTERN-config-layers and PATTERN-config-strictness: the retired keys go through configengine's `RetiredKeys`, not a local workaround.
- PATTERN-sandbox-coverage: reed stays exercised by the sandbox suites; a retired scenario names its reason.
- `render` stays the pure leaf its package doc declares: no I/O, no tmux, no engine import.
- No PATTERN entry changes; the Selvage rules in `internal/reedengine/doc.go` are module-local and are rewritten there.

## Auto-mode assumptions

- Q1–Q7 in the support log's ledger were picked without the operator; D1, D2, D3, D6 and D7 rest on them.

## Open risks

- A long-lived `lyx` process on the old binary (an orch, a shuttle run) keeps running the old stack layout and re-pins the `window-resized` hook on its reed ops until it ends; the next op on the new binary clears the hook and lays out nothing, so the window settles once the old processes are gone.

## Acceptance criteria

- `lyx reed add --cmd <c>` with no `--anchor` and no `--parent` opens the strand in its own window, and the shell window still holds only the Selvage pane.
- `lyx reed add --parent <guid> --cmd <c>` with no `--anchor` opens the strand in its own window and keeps its parent link, so a recursive remove of the parent removes it.
- `lyx reed add --anchor` with `below-parent`, `top`, `overview`, an unknown or an empty value exits non-zero with a message naming `own-window`, and adds nothing.
- `lyx shuttle run` accepts `own-window|hidden` and refuses `below-parent`.
- The engine refuses `AddStrand`, `AddStrandUnless`, `UpdateStrand` and `ReplaceStrand` with a retired anchor.
- A `reed.json` holding a strand with `anchor: below-parent`, a `size` object and `contentRows` loads and reads that strand as `own-window`.
- A live strand pane an older binary left in the shell window survives the first op on the new binary, ends up in its own agent window placed before the shell window, keeps its process, and its record reads `anchor: own-window`; a dead one is moved the same way and relaunches into that window; afterwards a relaunch leaves no second copy of the pane, and a remove leaves the shell window and a viewer on it in place.
- After a fresh boot, a Selvage heal (also with the Selvage corpse as the window's only pane) and a resize, Selvage is the shell window's only pane and fills it.
- A `reed.yaml` still holding `collapsed_rows`, `min_full_rows` and `selvage.height_rows` loads, and `lyx config reconcile` prunes them.
- Boot and attach clear a `window-resized` hook an older binary pinned on the shell window.
- `watchdog: off` still keeps the daemon's name repair off for that worktree.
- The status bar shows no per-strand pane buttons, and a server pinned by an older build has `M-z` unbound after the re-pin.
- Hidden strands still own no pane; own-window, group-window and overview placement behave as before.
- Outside the CLI refusal, the engine's refusal and read-time remap, retired-key fixtures, sandbox retirement notes and measurement reports under `docs/benchmarks/`, no `.go`, `.md` or `.yaml` file names `below-parent` or `AnchorTop`.
- CLAUDE.md holds no #673 workaround.

## Notes for the plan writer

- `internal/reedengine/apply.go`'s `planLayout` and `fixedHeightPins`, `internal/reedengine/selvagepane.go`'s `planPaneTarget` and `planTopPaneTarget`, and `internal/reedengine/reapply.go` are the main removal sites; `ownwindow.go`'s own `select-layout` for a shared group window is not part of the stack and stays.
- `spawn.go`'s `killReplacedOwnWindowPaneLocked` only kills a previous pane in an `@lyx_own` window, and a relaunch opens `-a` after the previous window; D3's move runs before either, so both hold for a migrated strand.
- The strands'-window resolver's "recorded strand pane placed in window 0" tier no longer fires once D3 has run; check its fallback with no Selvage and no window-0 strand.
- `internal/reedengine/retiredkeys_test.go` already pins a retired display-key fixture; the retired anchor and size keys can join it.
