# reed-view — decision record

## Goal

Make a reed run readable on a small screen: each strand viewable on its own as a zoomed tab with an overview to return to, every run reachable as a tab that never falls off a narrow terminal, and each strand role recognisable by its color in both tmux and Claude Code.

## Scope

In:

- A two-line tmux status bar for every reed session: line 1 holds the `VIEW` button, one button per strand and one per other window of the session; line 2 holds one button per run (session) and the clock.
- Alt-key navigation: Alt+z (zoom toggle), Alt+Up/Down (previous/next strand, zoom kept), Alt+Left/Right (previous/next run).
- Mouse clicks on the bar's buttons.
- Pane-border titles carrying the strand's label.
- A new verb `lyx reed switch --next|--prev` that Alt+Left/Right call.
- Session recreation in spawn order on the boot that spawns the hub's tmux server.
- Narrow-terminal handling of both bar lines (truncation and list-mode scrolling).
- A color per strand role, grouped by loom segment: on the strand's bar button and pane-border title, and in Claude Code's prompt bar through `/color`.
- `lyx ide spawn` seeding a marked `// lyx:begin` … `// lyx:end` block in the user's VS Code `keybindings.json`, so VS Code forwards the Alt keys.
- Retiring the single-line status-line pipeline the new bar replaces.

Out:

- The runs-overview window, the gate-checklist pane and any other new pane or window (notes `runs-overview`, `gate-checklist-pane`).
- One window per strand and tmux session groups (rejected in the operator's demo; note `reed-multi-window`).
- Changes to the strand layout itself (driver on top, spawned strands below, Selvage one line at the bottom).
- A keyboard binding for switching between a session's windows; tmux's prefix `n`/`p` and the line-1 window buttons cover it.

## Decisions

### D1 — Overview plus zoom

Decision: the window keeps reed's current layout as the overview, and a single strand is shown by zooming its pane (`resize-pane -Z`).
Rationale: chosen by the operator in the 2026-10-05 demo; a pane lives in one window only, so zoom is the only way to keep both the overview and a full-size single strand.

### D2 — Strand label and color as pane user options

Decision: reed sets two pane user options on every strand pane: `@strand`, the role segment of the strand's name as `internal/agentname` parses it (e.g. `driver`, `discussion`, `burler-review-2`), or the full name when it does not parse; and `@strand_color`, the tmux color of the strand's segment (D13), unset when the strand has none.
Reed sets both when it launches the strand's pane and re-asserts both on every bound, live strand pane in the attach pre-flight, so a resumed strand's new pane and a strand spawned by an older lyx get them too.
Selvage carries neither, and that absence is what keeps it off the bar.
Rationale: the pane title holds the long full name (`ly:reed-view:discussion`) and stays as it is; a short label needs its own option, and `agentname` is the sole parser of names (`PATTERN-agent-name`).

### D3 — Strand window marker

Decision: reed sets the window user option `@lyx_strands` on the strand window (the one `windowtarget.go`'s seam resolves), at boot and in the attach pre-flight.
The bar's formats find the strands through it, never through the current window.
Rationale: since `lifecycle-waits` the prime's session also holds `batten:<slug>` windows; marking the strand window keeps a batten window from being read as one, in the bar exactly as in reed's own targeting.

### D4 — Line 1: VIEW, strands, other windows

Decision: status line 1 is, left to right:

- `VIEW`, lit when the current window is the strand window and it is not zoomed.
- One button per pane of the strand window that carries `@strand`, in pane order, labelled with `@strand`; lit (background in the strand's color, black, bold) when that pane is the zoomed active pane, else unlit (strand's color on a dark grey).
  A pane with a wait mark appends ` ⏳<label> <elapsed>m` to its button, from `@lyx_wait` and `@lyx_wait_start` as the retired waits segment computed it.
  A strand with no `@strand_color` uses white.
- When the session has more than one window, one button per window not marked `@lyx_strands`, labelled with the window name, lit when current.

The line is drawn in tmux list mode (`list=on`, `list=focus` on the lit button, `<` and `>` markers), so an overflowing line scrolls and keeps the lit button visible.
Rationale: the operator's demo design, extended with window buttons because the prime now has batten windows and strand buttons that work from any window of the session.

### D5 — Line 2: runs and clock

Decision: status line 2 draws every session of the server with the `#{S:}` loop, in session-id order: the client's current session by its full name, lit (yellow, black, bold); every other session cut to 12 characters (`#{=12:session_name}`).
The run list uses tmux list mode with `<` and `>` markers and focus on the current session.
`%H:%M` is right-aligned after it.
Rationale: option 1 of the narrow-terminal note, recommended there; the current run is always visible and readable, the others stay identifiable, and overflow is marked instead of silently cut.

### D6 — Option pins and their scope

Decision:

- `status-format[0]` and `status-format[1]` are set globally (`-g`) only; tmux's status formats are one array, and a per-session index would drop the global lines for that session.
- `status` is pinned to `2`, session-scoped, replacing today's `on`; `status-position bottom` stays.
- `pane-border-status top` and `pane-border-format` (` <label> `, bold when active, in the strand's color, `selvage` for the pane without `@strand`) are window-scoped on the strand window only, so batten windows get no border title.
- Today's `status-left`, `status-left-length`, `status-right`, `window-status-format` and `window-status-current-format` pins are removed; the new formats do not render them.
- The key and mouse bindings of D8 and D9 are set server-wide.

All of these are issued at the boot that creates the session and again in the attach pre-flight, where `pinGeometryOptionsLocked` runs today, so a server booted by an older lyx picks them up at the next attach.
Like the geometry pins they are non-fatal: a failure is logged via `logger.Warn` and the session keeps working (Shared Decision geometry-tmux-failures-are-non-fatal-everywhere).
`mouse` and `remain-on-exit` keep their boot-only pins.
Rationale: the board placed the pins at server boot so they survive a restart; also pinning at attach covers servers that are already running when this lands, and the session/window scoping follows the existing rule that a scoped value wins over a global one.

### D7 — Geometry

Decision: the layout's row budget takes the status line's reserved rows from the `#{status}` readback (`2` reserves 2), as the code already supports, and subtracts the one row `pane-border-status top` adds at the top of the strand window.
The Selvage pane keeps its height; its border title rides the divider row it already has.
Rationale: reed sizes `select-layout` strings to the live window; an unaccounted row makes tmux silently rescale or overflow the layout (doc.go's silent-layout-rescale and Selvage-divider notes).

### D8 — Alt-key bindings

Decision: reed binds in the root table:

- `M-z`: toggle the zoom of the active pane when it carries `@strand` or the window is zoomed; otherwise nothing.
- `M-Up` / `M-Down`: move to the previous/next pane carrying `@strand` in the strand window, wrapping, keeping the zoom state (`select-pane -Z`); a pure tmux command sequence that steps over the Selvage pane; nothing in a window without `@lyx_strands`.
- `M-Left` / `M-Right`: `run-shell` of `lyx reed switch --prev` / `--next` with `--socket '#{socket_path}' --client '#{client_name}' --tmux <tmux path>`.

Bound: the bindings take these five keys from every pane on the hub's reed server, so a program inside a pane, Claude Code included, never receives Alt+z, Alt+Up/Down or Alt+Left/Right; no other key is bound.
Rationale: operator decisions of 2026-10-06; the step over Selvage stays reed-free because Selvage is always the one bottom-most pane without `@strand`.

### D9 — Mouse clicks on the bar

Decision: reed replaces the root `MouseDown1Status` binding with one that dispatches on the clicked range:

- `VIEW` (a user range): select the strand window and unzoom it.
- A strand button (a pane range): select that pane in the strand window and zoom it.
- A window button (a window range): select that window.
- A run button (a session range): switch the client to that session.

Bound: only `MouseDown1Status` is replaced; the right-click menus and every other mouse binding keep tmux's defaults.
Rationale: tmux's default selects the pane and so unzooms (the board's open item); making a strand button a "show this strand" tab is the coherent reading of a bar whose buttons are lit by zoom.

### D10 — `lyx reed switch`

Decision: a new reed verb `lyx reed switch --next|--prev --socket <path> --client <name> --tmux <path>`.
It lists the server's sessions, sorts them by numeric session id, finds the client's current session and switches that client to the next or previous one, wrapping at either end.
It resolves nothing from a working directory: no `lyxcwd`, no reed config, no reed state; every input is a flag.
On success it prints nothing, so `run-shell` opens no output view; a failure is a JSON error via `internal/output` and a non-zero exit, which `run-shell` shows.
Exactly one of `--next`/`--prev` is required.
Bound: the verb only ever runs `list-sessions`, `display-message` and `switch-client` on the told socket, against the told client.
Rationale: `switch-client -n`/`-p` step in name order while the bar draws in id order; the operator chose a lyx verb over a shell helper; flags keep it free of cwd resolution (`PATTERN-cwd-resolution`).

### D11 — The binding's `lyx` binary

Decision: the `M-Left`/`M-Right` binding names the absolute path of the `lyx` binary that pinned it, resolved through `panebin.go`'s chokepoint, with every shell token emitted through `internal/shell`.
`PATTERN-pane-binary-resolution` widens its scope from strand panes and the detached window to reed's key bindings that run `lyx`, in the same commit.
Rationale: `run-shell` must not resolve `lyx` from `PATH`, for the same reason a strand pane must not.

### D12 — Spawn order across a server restart

Decision: on a boot that finds the hub's tmux server not running, reed first brings up the session of every hub worktree that had one, in spawn order: the prime first, then each pair by its run's start time read from the pair's own records, a pair with no readable start time after the dated ones by worktree name; only then does it bring up its own session.
Each of those ups is that worktree's ordinary `up` (session and Selvage, no strand relaunch), under that worktree's own op lock, run sequentially and outside the booting worktree's op lock, so no two op locks are ever held at once.
The ordered list is told to `reedengine` by the hub-mode teller (the CLI layer); `reedengine` reads no loom or fabric record itself.
Standalone mode has no list and boots only its own session.
Bound: the step runs only when no server answers on the hub's socket, creates sessions only for worktrees whose reed state records a session, and starts no strand; an up that fails for one worktree is logged and skipped, and the boot goes on.
Rationale: session ids follow creation order, so creating the sessions in spawn order keeps D5's and D10's order across a restart (operator, 2026-10-06); concurrent booters iterate the same list, so each session is created only after its predecessor exists.

### D13 — Segment colors

Decision: a new leaf package (standard library only) declares one constant per loom segment and maps each to a provider-neutral color from Claude Code's `/color` palette, and each palette color to its tmux color:

| Segment | Color | tmux |
|---|---|---|
| Coordinator (driver, orch) | blue | blue |
| Discussion | purple | magenta |
| Plan | cyan | cyan |
| Webster (Merriam, card execution, recovery, rework) | green | green |
| Review (Burler, Bouncer, judges) | orange | colour208 |
| Describe | yellow | yellow |
| Landing (merge conflict, friction) | pink | colour205 |

Red is left unused, so it never reads as an error.
Each spawning module names its role's segment on the launch spec, next to the role and skills it already names; a strand with no segment has no color.
`PATTERN-leaf-packages` gains the new package with its closed import set.
Rationale: the operator wanted one color per segment, not only driver against the rest; the spawning module owns its role names (`PATTERN-agent-name`), so it owns the segment too, and one table keeps the colors themselves in one place.

### D14 — Where the color is applied

Decision:

- `shuttleengine.Spec` carries the segment, and shuttle passes it to reed's add spec; reed persists it on the strand record as an opaque display field and sets `@strand_color` from it (D2).
- After launch and before the skill load, shuttle asks the provider for its color command and types it; `claudeengine` answers `/color <name>`, and a provider without one answers empty, which types nothing.
- The orch is the exception: its color command is typed by the orch watcher, idle-gated, as the first step of its start and reload sequences, so a `/clear` reload restores it (`PATTERN-orch-pane-single-writer`).
- A failed color step is logged and never fails a launch: the color is display only, like the pane title.
- A new PATTERN entry records the rule: a spawned session's color is named as a segment on its launch spec, reed tints its tab and border and lyx types the provider's command; no stencil asks an agent to set a color.

Rationale: the board left open whether the spawning code or the agent's prompt sets the color; typing it from Go mirrors how skills are loaded (`PATTERN-role-skills-typed`) and keeps provider specifics in `claudeengine` (`PATTERN-shuttle-provider-seam`).

### D15 — VS Code keybindings block

Decision: `lyx ide spawn` writes, on every run, a `// lyx:begin` … `// lyx:end` block into the user's VS Code `keybindings.json` holding one `workbench.action.terminal.sendSequence` entry per key, `when: terminalFocus`: Alt+Up/Down/Right/Left as `\u001b[1;3A`–`D` and Alt+z as `\u001bz`.
An existing block is replaced in place; a file without one gets the block inserted before its closing `]`; a missing file is created holding only the block.
Nothing outside the markers is read for meaning or changed.
The path is the per-OS user directory: `$XDG_CONFIG_HOME/Code/User` (default `~/.config/Code/User`) on Linux, `~/Library/Application Support/Code/User` on macOS, `%APPDATA%\Code\User` on Windows.
A file that has no closing `]` outside a block, or a write that fails, leaves the file untouched; `lyx ide spawn` reports the skip and its reason in its result and still launches.
Bound: the verb writes only between its own markers, in that one file.
Rationale: operator decision of 2026-10-06, the same block shape mill uses for its own keys; VS Code has no per-workspace keybindings.

### D16 — Retire the single-line status pipeline

Decision: the new bar replaces the single-line status text, so its pipeline goes: the embedded `status-line.md` template, the `status_line.template` reed config key, `Engine.StatusLineText`/`ValidateStatusLine` and the boot validation, the `lyx reed statusline` verb, the `waits` token and placeholder swap, and `internal/tokenvocab`, whose only consumer is this pipeline.
`PATTERN-leaf-packages` drops `tokenvocab`.
An already-materialized `reed.yaml` that still carries `status_line` keeps loading, the key ignored.
Wait marks render on the strand buttons instead (D4).
Rationale: the operator's bar shows no identity text; the session's run tab already names the worktree, and the hub is one per server; keeping a config key and verb whose output is never displayed would be dead surface.

## Constraints

- `PATTERN-told-geometry`: `reedengine` derives no path and imports no `lyxcwd`; D12's ordered worktree list and its start times are told by the hub-mode teller.
- `PATTERN-cwd-resolution`: `lyx reed switch` resolves nothing from a working directory.
- `PATTERN-cli-cobra`: the new verb is a cobra command under `lyx reed` with a `Short` and JSON errors; the retired `statusline` verb leaves the tree.
- `PATTERN-pane-binary-resolution`: widened by D11 in the same commit.
- `PATTERN-shell-mechanics-seam`: the `run-shell` command string is built through `internal/shell` only.
- `PATTERN-agent-name`: `@strand` comes from `agentname.Parse`; nothing else splits a name.
- `PATTERN-shuttle-provider-seam`: `/color` and the Claude color names live only under `claudeengine`.
- `PATTERN-orch-pane-single-writer`: only the orch watcher types the orch's color command.
- `PATTERN-role-skills-typed`: the color is typed by lyx, never asked of an agent by a stencil.
- `PATTERN-wait-mark-display-only`: the strand buttons read `@lyx_wait` in a tmux format only; no Go decision reads it.
- `PATTERN-leaf-packages`: gains the color package, drops `tokenvocab` (D13, D16).
- `PATTERN-config-strictness`: an old `reed.yaml` carrying `status_line` still loads (D16).
- `PATTERN-sandbox-coverage`: the sandbox suite exercises `lyx reed switch` or excludes it with a reason.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: tests that need a tmux server are `tmux`-tier and run through `tmuxkit.Main`; no test reaches an LLM.
- `PATTERN-comment-line-breaks` and the semantic-line-break rule for Markdown.
- Docs in the same commit: `reedengine`'s and `reedcli`'s `doc.go`, the touched packages' `doc.go` (`shuttleengine`, `claudeengine`, `orchengine`, the ide packages, the new color package), `docs/overview.md` where the reed or CLI description changes, and `PATTERN.md` for D11, D13, D14 and D16.
- The bar requires tmux 3.6 features (`status-format` arrays with list mode and ranges, `range=user`, `#{mouse_status_range}`); the capability probe gains any verb the new pins need.
- Windows (psmux): the new pins, bindings and options are attempted unbranched and may be refused, the accepted degrade the status line already has.

## Auto-mode assumptions

- No operator answered questions in this session; every decision below the board's own was picked by this agent and is listed in the support log's question ledger.
- Window buttons on line 1 (D4) and `VIEW` working from a batten window: assumed wanted, since the prime gained batten windows after the board design was written.
- A click on a strand button zooms it (D9): assumed from the bar's buttons being lit by zoom.
- Retiring the single-line pipeline (D16) rather than showing its text on line 2: assumed from the operator's demo bar showing none.
- Segment-to-color table (D13): picked by this agent; the operator asked for a color per segment but named no colors.
- `lyx ide spawn` is the verb that seeds the keybindings (D15): the board says "as part of its setup", and `spawn` is the verb that writes VS Code setup today.

## Open risks

- Claude Code loses Alt+Left/Right (word jump) and any use of Alt+Up/Down and Alt+z in its input, since tmux takes them first; Ctrl+Left/Right still jump words.
- Whether Claude Code keeps a `/color` choice across `--resume` is unverified; a strand that reed resumes, not shuttle, may show the default prompt-bar color while its tmux tint stays correct.
- Typing `/color` right before the skill-load message assumes the immediate local command leaves the input ready at once; shuttle's launch readiness check is the guard to rely on.
- On WSL, VS Code reads the Windows-side `keybindings.json`, which lyx running inside WSL does not write; the seeded block lands on the Linux path and has no effect there.
- D12 brings back sessions in order only when reed itself boots the server; a server started by hand outside lyx gets sessions in whatever order they arrive.
- The `M-Left`/`M-Right` binding pins the path of whichever `lyx` last booted or attached; a dev binary that attaches to the hub's server rewrites it until the next attach by the prod binary.

## Acceptance criteria

- A reed session shows two status lines: line 1 with `VIEW`, one button per `@strand` pane of the strand window and, in a session with more than one window, one button per other window; line 2 with one button per session in session-id order and the clock.
- `VIEW` is lit exactly when the strand window is current and unzoomed; a strand button is lit exactly when its pane is the zoomed active pane; a window button is lit when its window is current; the current session's button is lit and shows its full name while other sessions show at most 12 characters.
- From a batten window of the prime, line 1 still shows the orch's strand button and `VIEW`, and the batten window's own button lit.
- Selvage never gets a strand button; its border title reads `selvage`; every strand pane's border title reads its `@strand` label in its segment color.
- A strand with a wait mark shows `⏳<label> <elapsed>m` on its button, and none once the mark is cleared.
- `@strand` and `@strand_color` are present on a freshly spawned strand's pane, on a resumed strand's new pane, and, after an attach, on a pane spawned before this change.
- Alt+z toggles zoom of the active strand and does nothing on Selvage or in a batten window; Alt+Up/Down step through the strand panes only, wrapping and keeping the zoom; Alt+Left/Right step through sessions in session-id order, wrapping.
- Clicking `VIEW`, a strand button, a window button and a run button each has its D9 effect.
- `lyx reed switch` with neither or both of `--next`/`--prev`, with an unknown client or an unreachable socket exits non-zero with a JSON error; a successful switch prints nothing; with one session it is a no-op success.
- A layout applied with the two-line status and `pane-border-status top` fills the window exactly, with no rescale and no overflow, at boot and after a resize.
- A server booted by an older lyx gets the new options and bindings at the next `lyx reed attach`.
- After the hub server is killed, the first boot recreates the prime's session first and the pairs' sessions in their runs' start order, before its own; a pair whose up fails is skipped and the boot completes.
- Each spawning module's launch spec names a segment; the strand's `@strand_color` matches D13's table, and shuttle types the provider's color command before the skill load; a provider with no color command types nothing; a failed color step does not fail the launch.
- The orch's color command is typed only by the orch watcher, at start and at each reload.
- `lyx ide spawn` inserts the block into a `keybindings.json` without one, replaces an existing block in place, leaves everything outside the markers byte-identical, is idempotent on a second run, creates the file when missing, and reports a skip without failing on a file it cannot place the block in.
- An existing `reed.yaml` carrying `status_line` loads; `lyx reed statusline` no longer exists; nothing imports `internal/tokenvocab`.
- `go test ./...` and the `tmux`-tier tests pass.

## Notes for the plan writer

- The strand window seam is `windowtarget.go`; the existing pins live in `pinGeometryOptionsLocked` (`windowsize.go`), called from the boot path in `lifecycle.go` and from the attach pre-flight in `attach.go`.
- Strand panes are created in `spawn.go`, which already sets `allow-set-title off` and the pane title; that is where `@strand`/`@strand_color` join.
- tmux 3.6's own default `status-format[1]`/`[2]` (visible with `tmux show -g status-format` on a server with `status 3`) show list mode with `#{P:}` and `#{S:}` loops and `<`/`>` markers, a working reference for D4 and D5.
- `#{S:}` iterates in session-id order by default; `#{S/n:}` would sort by name and is not wanted.
- Claude Code's color names map to tmux as: purple→magenta, orange→colour208, pink→colour205, the rest by the same name; Claude Code itself uses this mapping for its own tmux panes.
- Role constants sit in their spawning modules (`grep -rn 'Role *= *"'` under `internal/`); the driver's spec is built in `internal/loomcli/driverspec.go`, the orch's reload sequence in `internal/orchengine`.
- The operator's own `keybindings.json` already holds a hand-written, unmarked copy of the five entries; D15 leaves it alone, and duplicate entries are harmless to VS Code.
