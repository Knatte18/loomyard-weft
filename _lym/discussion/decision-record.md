# Decision record — configurable action buttons on the reed status bar

## Goal

Give every reed session a row of clickable action buttons, defined in config rather than code, that run shell commands on the run the operator is watching: halt, cancel, open in VS Code, another terminal, a new agent, and orch distill and refresh.
Adding a button is one config entry and needs no build.

## Scope

In:

- A third status line holding the action row; the `follow` button moves there as a built-in entry.
- A `buttons` key in reed's config, grouped by place (`task`, `beam`, `prime`), with the shipped default set.
- Machine additions through a `buttons` key in `desktop.yaml`'s machine layer.
- Placeholder filling, the optional `confirm` prompt, the per-agent-window buttons, click dispatch, failure display and output logging.

Out:

- The board-agent button of note `board-agent-button`: it becomes one more config entry once its skill and launch exist.
- Any change to the window buttons (line 1) or the session buttons, check mark and clock (line 2) beyond removing `follow` from line 2.
- A prompt that asks the operator for a value at click time.

## Decisions

### D1 — A third status line holds the action row

- Status lines: line 1 window buttons, line 2 session buttons then, right-aligned, the check mark (`Engine.SetCheckMark`) and the clock, line 3 the action row.
  The status pins set `status` to 3 on the session and `status-format[2]` with `-g`, beside the existing two, since a per-session index would hide the global lines.
- `follow` leaves line 2 and is the first entry of line 3 on every session, standalone included; its range, click branch and look are unchanged.
- The row is left-aligned in tmux list mode with the `<` and `>` markers, so a row wider than the client scrolls like the other two lines.
- Line 3 is the same in every window of a session, except the agent-window buttons of D5.
- The global status-format pins join the bindings' stale-build re-pin (`repinStaleBindingsLocked`), so the first reed op of the newest binary restores them after an older binary pinned its own.
  Bound: between an older binary's boot or attach and the newest binary's next op, `follow` can show on lines 2 and 3 and button clicks do nothing; any session an older binary boots or attaches keeps `status 2`, without line 3, until the newest binary next attaches it.

Rationale: the operator's placement (2026-10-30) keeps actions apart from navigation.

### D2 — The buttons are a `buttons` key in reed's config, grouped by place

- `reed.yaml` gains `buttons` with three groups, `task`, `beam` and `prime`, each a list of entries in display order.
  An entry has `label` and `run`, both required, and the optional `confirm` (prompt text) and `window` (only value `agents`).
- A session takes the group of its place (D4).
  The prime takes `prime` when that list is non-empty, else `beam`; the template ships `prime: []`.
- The default set (D8) ships in reed's template; under `PATTERN-sparse-hub-base` it is the hub's default, and the hub's base `reed.yaml` holds only the groups it tunes.
- The key is classified `CategoryExecutable`, since it runs commands: the hub's base and a worktree file may hold it, the local layer refuses it.
- An entry is valid when `label` and `run` are non-empty, `label` and `confirm` hold no `#`, no `%` and no control character, `window` is empty or `agents`, and `{strand}` appears only in an entry with `window: agents`.
  Validation runs at render (D5), for reed's and the machine's entries alike: an invalid entry is skipped with one Warn naming the file, group and index, and the rest render.
- Reed's and desktop's `Config` types decode `buttons` as an untyped YAML node that `LoadConfig` never interprets.
  The render decodes each group and entry, and skips one of the wrong YAML type with the same Warn as any other invalid entry, so a type error the layers admit stays inside the button row and leaves whole every load of `reedengine.LoadConfig` and of `desktophook.LoadConfig`, whose `afterWindow` runs on every window lym opens.
  A `buttons` key that is not a mapping (null, list or scalar) fails that config's load, as any key of the wrong shape does.
  A group of any other shape, a mapping included, loads in reed's hub base and worktree files and in desktop's hub file with no load-time Warn, and the render skips it with the invalid-entry Warn; so does an empty mapping in desktop's machine layer.
  In desktop's machine layer a group written as a non-empty mapping is refused as a key the template lacks, so desktop's load fails, adds no machine buttons (D3) and fails that load's `afterWindow` report.
  Beyond that shape, no reed verb fails on a button.

Rationale: the operator's design (2026-11-01) makes buttons config in reed's file; classifying by what the value can do follows `PATTERN-config-layers`.
Barring `#` keeps a label or prompt from injecting tmux format or style.
Barring `%` keeps it from drawing a time field: `#{E:}` applies strftime to the option value (tmux 3.6 probe: an option holding `L%Hx` expands to `L13x`), and the `confirm-before` prompt takes status-left sequences too.
A bad button must not stop agent spawns, so it costs only itself, except a machine-layer group written as a non-empty mapping, which costs desktop's whole load.

### D3 — A machine adds buttons through `desktop.yaml`'s machine layer

- `desktop.yaml` gains `buttons` with the same three groups and entry shape, template `task: []`, `beam: []`, `prime: []`, classified `CategoryMachine`, so it is set in `<BoltDir>/.lym/config/desktop.machine.yaml`, by hand or `lym config desktop --layer machine --edit`, and the local layer refuses it.
  Each group is one list unit, so the machine layer's shape check, mapping or not, admits a non-empty list against the template's empty one (`yamlengine.LeafPaths`).
- A session's row is reed's group for its place (D2), then the machine's: the machine's `prime` list when non-empty, else its `beam` list, for the prime.
- A failed `desktop` load is logged at Warn and adds no machine buttons.

Rationale: the machine layer exists only for hub-wide modules, and `desktop.yaml` is the hub-wide module whose keys are facts of the machine's desktop; a button a machine adds to its own status bar is such a fact, so no category or PATTERN entry changes.

### D4 — The session's place and placeholder values come from the hub geometry, lazily

- `hubgeom.ReedGeometry` tells the engine a lazy closure, like `SpawnOrder`, that the render (D5) calls; building the geometry stays free of reads.
  The closure returns the place, the three cheap values `{slug}`, `{worktree}` and `{session}`, the machine's `buttons` (D3), and a separate function that resolves `{beam}`.
- Place: `prime` for the prime geometry; `beam` for a pair whose origin record carries `Beam`; `task` for every other pair.
- Values: `{slug}` the worktree name (the prime's own name in the prime); `{worktree}` the session worktree's anchor path, the geometry's `AnchorPath`, which `PaneCwd` already equals; `{session}` the tmux session name; `{beam}` the beam's anchor path: the prime's in the prime, the beam's own in a beam session, and for a task the anchor of the worktree `hubgeom.ParentOf` names over the prime's location, read through `lymcwd.NamedWorktree`'s `AnchorPath`, the prime's when `ParentOf` returns `""`.
- The render calls the `{beam}` function only when an entry of the composed row uses `{beam}` or the session is a task, whose run directory it is (D7); the cost, one worktree listing under the op lock at boot and attach, is accepted.
- When the worktree `ParentOf` names has no anchor directory on this machine, every entry using `{beam}` is left out of the session's row with one Warn naming the entry and the missing beam.
- A failed desktop load inside the closure is logged at Warn and returns no machine buttons, leaving the place and values intact (D3).
  Only a failure to read the place or a value, such as an unreadable origin record, is logged at Warn and shows `follow` alone.
- A standalone session has no closure and shows `follow` alone.

Rationale: reedengine gets absolute values and never resolves a location (`PATTERN-cwd-resolution`); reading a task's parent goes through `ParentOf` (`PATTERN-pair-parent`).
The path values are anchors, not worktree roots, because `lymcwd.Resolve` requires the cwd to equal `AnchorPath` exactly, and an anchor may be a subpath recorded in `.lym-anchor`; a lym verb run in the worktree root of a subpath-anchored hub is refused with `ErrCwdOutsideAnchor`.
`ParentOf` names a beam pair even when it is absent on this machine, and `NamedWorktree` builds its path without checking that it exists, hence the presence check.

### D5 — The row is rendered into session options at the status pins

- At boot and in the attach pre-flight, where the other status pins run, reed validates (D2) and renders the session's entries into four session user options, set in one tmux invocation of chained commands so no click sees a row and a spec from different renders:
  - a row for windows whose active pane carries no `@strand`, holding the entries without `window: agents`;
  - a row for windows whose active pane carries `@strand`, holding every entry in config order;
  - the button spec: the absolute path of reed's configured shell, the run directory (D7), and per entry its index, label, `run` with the static placeholders filled (D6) and `confirm` filled the same way;
  - `@lym_button_dir`, the session's run directory (D7).
- `status-format[2]` is the follow button then `#{?#{@strand},<agent row>,<plain row>}` over those options, each row expanded with `#{E:}`.
- Entry `N` is drawn as a user range named `lymbtn<N>`, at most 15 bytes as tmux requires for `range=user|X`, in the unlit button style.
- The two row options stay display only, as `@strand` does.
  The spec option, `@lym_button_dir` and the pane option `@lym_strand_name` of D6 are decision-bearing, beside `@lym_follow`; the bar's and pane options' doc lines that name `@lym_follow` as the one such option change with them.
- A config edit, or a pair marked a beam after boot, reaches a live session at its next attach or boot.

Rationale: tmux's status formats are one global array, so per-session content must come from session options.
tmux expands `status-format` first and draws the result, so ranges and styles that come out of `#{E:}` are drawn and clickable (tmux 3.4 to 3.6 `status.c`: `format_expand_time`, then `format_draw`).
Holding the spec beside the drawn row means a click runs the command drawn under it, even after a config edit.

### D6 — Placeholders are filled and shell-quoted per value

- The placeholders are `{slug}`, `{worktree}`, `{beam}`, `{session}` and `{strand}`; any other brace text, such as an awk program, stays literal.
- In `run` each value is substituted quoted for reed's pane shell dialect through `internal/shell`, so a path holding spaces stays one word.
  In `confirm` it is substituted plain with each `#` and each `%` doubled, so a value cannot inject a status sequence or a time field into the prompt.
- The four static values are filled at render (D5); `{strand}` is filled at click time.
- `{strand}` is the full strand name of the clicking client's active pane, read from a new pane option `@lym_strand_name` that reed sets with the full name wherever it sets `@strand`, which holds only the role.

Rationale: quoting per value keeps a config author from having to quote, and makes no value able to break out of its word.

### D7 — A click runs `lym reed button`, which confirms, runs, logs and reports

- The left-status-click dispatch gains one branch for ranges matching `lymbtn*`.
  It runs `run-shell -b -c '#{@lym_button_dir}'` with a line composed like `composeFollowCommand`: the absolute path of the binary that pinned the bindings, `reed button`, the socket, the tmux path, the session, the clicked range and the client, with the verb's stdout and stderr discarded.
  The pinned line, on the click line and the confirmed line alike, ends with a forced zero status in reed's pane shell dialect, the dialect it is composed in (`; exit 0` in POSIX sh, the equivalent in pwsh).
- The run directory is the beam's anchor (`{beam}`) in a task session, the prime's anchor when that beam is absent on this machine (D4), and the session's own anchor elsewhere, so the default CANCEL needs no `cd` (D8).
  The verb's log therefore lands in that anchor's logs.
- `lym reed button` exits 0 on every path, on the click line and the confirmed line alike: a changed spec, no `@lym_strand_name`, an unreadable spec option, a gone run directory, a failed command, and a `display-message` that itself fails.
  tmux 3.6 opens view mode in the target pane for a background `run-shell` command that exits non-zero even with no output (probe: `run-shell -b` with `exit 3 >/dev/null 2>&1` leaves the pane in view-mode).
  The discarded output and the forced zero status together keep view mode shut, whatever the binary or verb does: a removed or replaced pinned binary, a panic and a kill signal included.
- `lym reed button` is a role-audience verb under the reed cobra subtree, like `reed follow`; it reads no config and no reed state, only the session's spec option and, for `{strand}`, the client's active pane's `@lym_strand_name`.
- An entry with `confirm`, clicked without `--confirmed`, issues `confirm-before -t <client> -p "<confirm> (y/n)"` whose command re-runs the same line with `--confirmed`, a hash of the entry's spec `run` and, for a `{strand}` entry, the strand name read at the click.
  The confirmed run reads the entry from the spec again and refuses, with a status message saying the buttons changed, when its `run` no longer hashes the same; no `run` text crosses tmux's parsing.
  The carried strand name must pass `agentname`'s name grammar.
- A `{strand}` entry clicked while the active pane carries no `@lym_strand_name` shows a status message and runs nothing.
- The verb runs `run` through the spec's shell in the spec's run directory, and waits for it with no timeout.
  When that directory is gone, as when a task's beam is removed after the render, it shows a status message naming the directory and saying the row re-renders at the session's next attach, and runs nothing.
  It sets no caller identity: a click already reads as the operator because reed strips `agentname.StrandNameEnv` and `agentname.ParentEnv` from the tmux server's environment at spawn (`stripAgentNameEnv`), and a direct call from a strand that runs the entry itself keeps that strand's identity.
- Spawn, exit code and combined output go to lym's log at Info.
- A failed spawn or a non-zero exit shows `display-message -c <client>` with the label, the exit code and the first output line; a client gone by then, as after CANCEL, only fails that message, logged at Debug.
- The command is the operator's action, not a run's process: it is not registered in `internal/runproc`, and the commands it starts register their own processes as they do from a terminal.

Bound: a click runs only the command the session's spec holds at that index.
Any process that can run `lym reed button` against the socket can run a spec entry, `--confirmed` skipping its prompt, under its own caller identity, which adds nothing over running the command itself.
A direct call without `--confirmed`, on an entry with `confirm`, raises the prompt on whichever client it names, and a `y` there runs the entry as the operator, since the confirmed line runs from the tmux server; this adds nothing over the socket, which can already run commands through `run-shell`.
A process that can write a file that defines buttons, `reed.yaml`'s hub base and worktree layers or `desktop.yaml`'s hub base and machine layer, or reach the tmux socket, an agent pane among them, can change what a button runs, and so run a command as the external caller on the operator's next click; such a process can already run commands itself, so the added reach is the caller identity alone, accepted.
A click whose pinned binary is gone runs nothing, logs nothing and shows no message, until a reed op of another build, not older than the pinned one, re-pins the bindings (`bindingsStale`).
Two clicks run the command twice; irreversible entries carry `confirm`.

Rationale: tmux cannot index a per-session command by a clicked name in one global binding, so a verb resolves it; re-invoking after `confirm-before` keeps output logging and failure display on the confirmed path.

### D8 — The shipped default set

- `task`:
  - `HALT`: `lym tender pause {slug}`
  - `CANCEL`: `lym batten cancel {slug}`, confirm `Cancel {slug}?`
  - `code`: `lym ide spawn {slug}`
  - `term`: `lym reed window {worktree} --title {slug}`
  - `agent`: `cd {worktree} && lym shuttle open --role agent --model 'opus[high]'`
- `beam`: `code`, `term` and `agent` as above, then `distill`: `lym orch distill`, confirm `Distill the orch?`, and `refresh`: `lym orch refresh`, confirm `Refresh the orch?`.
- `prime`: `[]`, so the prime takes the `beam` set.
- The POSIX and Windows templates both ship the set; `cd <dir> && <cmd>` runs under bash and pwsh 7 alike.

Rationale: the entries the operator named.
`lym tender pause` holds agent and network work and interrupts running turns at once, unlike `lym batten pause`.
`lym batten cancel` runs only from the prime or a beam (`ResolveBattenPrime`); a task session's commands run in its beam's anchor (D7), so CANCEL needs no `cd`, while `{worktree}` still names the task for `term`.
`term` passes `{slug}` as its title, since the window title defaults to the directory's base name, which in a subpath-anchored hub is the anchor's and the same for every worktree.
`agent` changes to `{worktree}` first, since `lym shuttle open` opens the session in the worktree it runs from; in a beam or the prime the `cd` is a no-op.
CANCEL on a task whose beam is being landed is refused by `ResolveBattenPrime`, which names the worktree to re-run from, and that refusal reaches the operator as D7's failure message.
The model spec is quoted because `opus[high]` is a bash glob.
The new-agent button opens a fixed role, `agent`, since asking for a role needs a prompt nobody asked for.

### Added after Discussion (parent, 2026-11-01): A mapping group in desktop's machine layer fails that load, empty or not; Discussion accepted at the hard cap

Decision: D2 is corrected: in desktop's machine layer a buttons group written as a mapping, an empty mapping included, is refused by ValidateMachine's shape check (internal/configengine/layers.go, the template holds a different shape), so desktop's load fails, adds no machine buttons and fails that load's afterWindow report. The clause 'so does an empty mapping in desktop's machine layer' is dropped and 'non-empty mapping' reads 'mapping' where D2 and its rationale say it. Plan verifies this against ValidateMachine and writes a test for the empty-mapping case. The other round-2 fixes stand: D7's pinned click and confirm line ends with a forced zero status, task commands run in the beam's anchor with {worktree} naming the task, and the new-agent button's fixed role is agent (D8).

Rationale: Parent, 2026-11-01: the driver read layers.go and found the empty-mapping claim false; recording the correction here lets the segment be accepted at its hard cap without carrying a known false statement into Plan.

## Constraints

- tmux floor 3.4 (`toolfloor.TmuxFloor`); every construct used, `range=user|X`, `#{E:}`, `#{m:}`, `confirm-before -p`, `run-shell -b -c`, exists there.
- psmux on Windows: the new pins and binding are attempted unbranched and non-fatal, like the existing status pins.
- `PATTERN-shell-guard-table` and `PATTERN-spawn-observability` bind D6 and D7.
- `PATTERN-config-layers`: both new keys are classified in their module's `ConfigSpec`; `reedengine` and `desktophook` stay degrading callers.

## Auto-mode assumptions

- `{beam}` is an anchor path, not a slug (D4).
- The new-agent role is fixed to `agent`, settling the entry's open question (D8).
- `follow` is first on line 3 (D1).
- Config edits reach a live session at its next attach or boot (D5).
- The beam set also carries `code`, `term` and `agent` (D8).
- A task session's button commands run in its beam's anchor, the prime's when the beam is absent here, not in the task worktree the board entry names, so the shipped CANCEL works without a `cd`; `{worktree}` names the task for commands that need it (D7).

## Open risks

None.

## Acceptance criteria

- D1: a booted or attached session shows three status lines; line 2 has no `follow` button and keeps the check mark and clock; line 3 starts with `follow`, whose click still toggles the follow state; after an older binary re-pins `status-format[1]`, the newest binary's next reed op restores it.
- D2: the template's `buttons` loads; each invalid shape D2 names, a group or entry of a wrong YAML type the layers' shape check admits included, is skipped with a Warn while the other entries render, a group written as a mapping in `reed.yaml` included, which loads and is skipped at render, `reedengine.LoadConfig` and `desktophook.LoadConfig` still load and `lym reed up` succeeds; an empty `prime` list gives the prime the `beam` entries and a non-empty one replaces them.
- D3: an entry in `desktop.machine.yaml` appears after reed's entries of the same place; the local layer refuses `buttons` in `desktop` and in `reed`.
- D4: the prime, a beam pair and a task pair each get their group; a task's `{beam}` is its parent beam's anchor path, or the prime's when it has none; a task whose parent beam is absent on this machine leaves out its `{beam}` entries with a Warn; a failed desktop load drops only the machine buttons; a standalone session and a failed place or value read show `follow` alone.
- D5: a window whose active pane carries `@strand` shows the `window: agents` entries and other windows do not; ranges are named `lymbtn<N>`; after a config edit an attach re-renders the row.
- D6: placeholder values holding spaces reach the command as one word; unknown brace text stays literal; `{strand}` fills with the active pane's full strand name; a `#` or `%` in a value shows literally in the confirm prompt.
- D7: a click on an entry without `confirm` runs its command in the session's run directory with the strand-name variables unset; a `confirm` entry, `{strand}` ones included, runs only after `y` and refuses when the spec changed in between; a failing command shows a status message and its output lands in the run directory's log; no click, refused or failed included, opens a view-mode pane.
- D8: both templates hold the task, beam and prime groups as listed; CANCEL clicked in a task session whose parent beam is present on this machine and not being landed runs `lym batten cancel` from that beam's anchor without the location refusal.

## Notes for the plan writer

- The follow click branch and `composeFollowCommand` are the model for the button branch and its pinned line.
