# runs-overview — decision record

## Goal

Give the operator one place in the prime's reed session that shows every run in flight and the loom stage it is in, without polling each run or reading the orch's chat.
Build with it the board entry's merged sections listed under Scope.

## Scope

In:

- A runs overview strand directly above the orch strand in the prime's strand window, one line per batten run in flight, its height following its line count.
- Strand sizing in reed: a top anchor (to put the overview above the orch whatever the insertion order), a content-sized kind and a sharing kind beside today's positional rule, which stays the default.
- Seat-table agents that run side by side (the seats engine's chair and advisors) declared as sharing strands.
- A gate checklist strand in every task pair's reed session, for both child drivers.
- A hub notice log that batten appends every notice to.
- Selvage in the Alt+Up/Alt+Down cycle, and Alt+z zooming Selvage.
- The `lyx batten run --window` terminal window closing when its run ends, on Konsole and on Windows Terminal under WSL.

Out:

- Columns, side-by-side panes and one tmux window per strand: every kind stacks in rows.
- Alt+Up and Alt+Down stepping between windows (the board entry parks it for later).
- Showing the notice log in the overview pane.
- Any change to what a batten notice says or to whom it goes.
- A `lyx reed` CLI verb for sizing.
- Who owns the overview after `orch-split`; until that note is built, the orch start owns it (D1).

## Decisions

### D1 — The overview is a batten pane verb in a strand the orch start ensures

`lyx batten overview` is a new hidden verb (internal audience, cobra subtree of `batten`, JSON errors per `PATTERN-cli-cobra`) that runs in a pane and redraws the run lines.
`lyx orch start` ensures a strand named `runs` running it in the prime's reed session before it ensures the orch strand; on every start it replaces a dead `runs` strand and a live one launched by another lyx build, the way the loom bootstrap replaces a `loom-status` strand from an older build.
`lyx orch stop` leaves it.
The strand names no segment and gets no color (`PATTERN-spawn-color`).
Its size declaration is in D5.
Rationale: batten owns the runs and their status files; the orch start is the one verb that builds the prime's strand window; the overview stays useful while the orch is stopped.

### D2 — Which runs show, and what one line shows

A run is in flight when its prime batten run directory's status state is anything but `done`, and its pair worktree exists or its current producer is Worktree-Create.
Halted and failed runs with a pair still show; a run leaves the list when it reaches `done` or its pair is gone.
Lines are ordered by the seed's start time, and runs with no readable seed come last, by slug.
One line per run: the slug, the batten state, the loom stage, the time in that stage, the run's age, and the pair's reed session name.
The loom stage is the pair's loom child's current producer while batten is in its Run-Shed row, and batten's own current producer otherwise, also when the child's status file does not exist yet or any more.
A status file that fails to read or decode on two polls in a row shows the slug and `status unreadable`; one failed poll keeps the previous line; neither stops the other lines.
No run in flight renders one line, `no runs in flight`.
The pane is redrawn in place, never scrolled, and only when a line's text changes; it polls at most once a second, backs off while nothing changes (`PATTERN-daemon-wakeups`), and re-decodes only a status file whose modification time changed.
It reads status files without the run's status lock, which is safe because `internal/state` writes them atomically; it never blocks or slows a run.
This departs from the board entry's note that the overview carries each run's attach command (handed over from `batten-notice-trim`): an attached operator reaches a run by its session name through Alt+Left/Alt+Right or the session buttons, and an operator outside the reed session gets the full command from the notice log (D8).

### D3 — Strand sizing in reed

`render.Display` gains an optional size declaration persisted with the strand in reed.json, and the anchor vocabulary gains `top`; `own-window` stays reserved and refused by Rules.

- **Positional** (no size declared): today's rule; every strand that declares nothing keeps today's layout.
- **Content**: a preferred share of the free height, a minimum in rows, and a maximum equal to the strand's current content rows.
  Its height is `min(content rows, max(min rows, floor(share × free height)))`.
  Content rows count visible content lines; reed adds the title row a pane at window row 0 loses to `pane-border-status top`, so a pane of N content rows shows N lines.
  Until its first report (D4) a content strand's content rows are its minimum.
- **Sharing**: the strand splits the rows left after the fixed-height panes evenly with the other live sharing strands of the window.
  While any sharing strand is live, every positional strand takes `collapsed_rows`; with none live, the bottom-most positional strand expands as today.
  Sharing and positional strands keep one below-parent stack in insertion order.

Free height is the strands' window's render box less the Selvage band.
Top-anchored strands stack above the below-parent stack, in insertion order.
Fixed-height panes are Selvage, the collapsed positional strands and every content strand; the window-resized hook pins every one of them, as it pins Selvage today.
A window too short reclaims rows from collapsed placements first, then from content strands down to one row each, then from the expanding strands, and every pane keeps a positive height.
The spawning module sets the declaration on its add spec (`reedengine.AddSpec`; shuttle's launch spec carries it inside `Display`); reed reads it only to place panes.
A strand's removal stays its spawner's, as today.
Rationale: it builds the operator's 2026-10-12 sizing rule and the 2026-10-08 two-kind proposal on the one layout function, and leaves every existing caller's layout as it is.

### D4 — A content strand reports its line count to reed

reedengine gets one engine operation, told a strand name, a pane id and a count, that sets the strand's content rows, persists them, and re-applies the layout under the zoom bracket (`keepZoomLocked`).
The pane process calls it with its own `$TMUX_PANE` whenever its line count changes, and retries at its next poll after a refusal.
Bound: it refuses a strand that does not exist, declares no content sizing, or whose recorded pane is not the told pane, and a count below 1; it changes nothing but that strand's content rows; an unchanged count re-applies nothing.
So a hand-run `lyx batten overview` or a replaced process can resize no strand.
When the content holds more lines than the pane shows, the pane fills all rows but the last with lines and ends with a summary line naming how many are not shown; the pane process reads its own height from its terminal, so the summary is right before reed has applied a new count.
The overview drops the newest runs first; the checklist keeps its state line and the running row in view and drops rows furthest from the running row first.

### D5 — Sizes in use

- The `runs` strand: top, content, share one third, minimum 3 rows (the operator's numbers).
- The gate checklist strand (D7): top, content, share one quarter, minimum 3 rows; a smaller share because a pair window also holds the driver and the agent panes.
- Every strand a seats-engine seat table launches (chair and advisors): sharing.

Every other strand declares nothing and keeps the positional rule.
The shares and minimums are constants in code, not reed.yaml keys.

### D6 — Seat tables declare sharing through the existing launch path

The seats engine sets the sharing declaration on each seat's shuttle launch spec; shuttle passes `Display` to reed unchanged.
Rationale: the spawning module sets its strands' kind, as the board entry's open point proposed, and reed stays a dumb carrier.

### D7 — The gate checklist is the pair's status strand

`lyx loom` gets a hidden pane verb that renders the run's recipe rows, read from the run's seed recipe through `shedbuild` (`PATTERN-sole-parsers`), as a checklist.
The first line is the run's state line through `shedverbs.RenderStatusLine`.
Then one line per row in recipe order: a box (done, running, failed, not reached), the row name, the time spent in it summed over its visits (a history entry's start to the next entry's start, and to now for the running row), and the visit count when above one.
History entries naming a row the recipe no longer has follow the recipe rows, marked as such.
The loom start bootstrap runs it in the existing `loom-status` strand, keeping that strand's name, its build sidecar and its keep-or-replace logic, and keeps the strand on both drivers instead of removing it on an llm-driven run.
The strand names no segment.
It reads the status file the way D2 does, and takes its height from D3, D4 and D5.
Rationale: one status pane per pair; the checklist holds the one-line status and more; the llm driver is the default child driver, so a checklist on go-driven runs only would almost never show.

### D8 — The hub notice log

Every batten notice queued or logged for the orch is also appended, once per delivered notice and never per attempt, to one hub log file under the prime's `.lyx`: one line with the UTC time, the notice text and the run's attach command, `cd <pair dir> && lyx reed attach`, the directory quoted through `internal/shell`.
Each line is one `O_APPEND` write, so concurrent batten processes never interleave inside a line.
The path is declared through `internal/lyxdirs` and `internal/shedrun`, resolved in `battencli`'s wiring and told to `battenshed` through its deps (`PATTERN-told-geometry`), so `battenshed`'s import allowlist is unchanged.
A failed append is logged and never blocks or changes the notice.
The log is append-only and never rotated: a notice goes once per condition episode, so it grows by a few lines per run.
`lyx batten`'s help text names the log's path.

### D9 — Selvage joins the strand-pane cycle

Alt+Up and Alt+Down step through the strands' window's strand panes and Selvage as one cycle in physical order, Selvage last, wrapping, keeping the zoom, so a zoomed step onto Selvage shows it full screen.
Alt+z zooms and unzooms Selvage as it does a strand pane.
A hidden batten window's pane stays outside both: Alt+z and the steps act only in the strands' window.
The bindings find Selvage through a display-only pane marker that no Go decision reads (as `PATTERN-wait-mark-display-only` holds for the wait mark), set where reed creates Selvage and re-asserted in the attach pre-flight; Selvage still never gets `@strand` or `@strand_color`.
A zoomed Selvage stays zoomed through every layout re-apply, under the existing zoom bracket.

### D10 — The run terminal window closes when its run ends

The window's shell line becomes: run the attach; on exit status 0, end the window; on any other status, print the status and that the window waits up to 30 seconds for the run's worktree to be removed, end the window if the pair directory disappears in that time, and otherwise keep an interactive shell.
An operator detach (status 0) closes the window too.
The same line serves Konsole and Windows Terminal, built through `internal/shell` (`PATTERN-shell-mechanics-seam`).
Bound: the window ends only on a 0 attach or on the pair directory gone within the wait; every other ending keeps the shell and the attach's output.
A pair session ended while its run lives (`lyx reed down`, a watchdog reap) also ends the attach with 0 and closes the window; the run stays reachable from the prime by session switch.
Rationale: a pair's session ends at Worktree-Teardown, before its worktree is removed, ending the attach with 0.

## Constraints

- `PATTERN-told-geometry`: reedengine and render take every size and path they are given; the overview and checklist renderers take status paths, recipe rows and the pane height as values, and the CLI layer resolves them (`PATTERN-cliwire-sole-wiring`, `PATTERN-cwd-resolution`).
- `PATTERN-leaf-packages`: `render` stays a pure leaf; the size math lives in its policy layer, never in its mechanics layer.
- `PATTERN-shed-run-directory`: run directories, status and seed paths only through `shedrun`.
- `PATTERN-sandbox-coverage`: no new module is registered, so the sandbox suite needs no change.
- The positional rule's output for a window whose strands declare nothing is byte-identical to today's.
- A strand persisted by an older lyx, with no size declaration, lays out positionally.
- Docs change in the same commit: reedengine's `doc.go` layout-rule and bindings paragraphs, `render`'s anchor vocabulary comment, `orchengine`'s, `battenshed`'s and `loomcli`'s docs, and `termwindow`'s package comment.

## Auto-mode assumptions

- A1: "age" on a line means both the time in the stage and the run's age (D2).
- A2: the overview's attach command moves to the notice log (D2, D8).
- A3: the checklist takes over the `loom-status` strand, and now shows on llm-driven runs at a quarter share (D5, D7).
- A4: "gates" in the checklist means the loom recipe's rows (D7).
- A5: shared strands stack in rows, though `reed-multi-window`'s 2026-10-05 note prefers one tmux window per strand on a small screen; the board entry's 2026-10-08 sizing proposal leaves rows open, and the depth idea is that note's own work.
- A6: an operator detach closes the run window (D10).

## Open risks

- tmux's attach exit status: a session killed while attached is expected to end the client with 0 and a server shutdown with non-zero; D10's directory wait covers the second.
  The plan verifies both on the installed tmux.
- An operator `tmux_conf` setting `detach-on-destroy off` moves the run window's client onto another session when the pair session ends, so the window stays open showing that session.
- The window-resized hook array grows by one pin per content strand; the resize behaviour measured for Selvage is assumed to hold for more pinned panes.
- Seat agents sharing rows on a small laptop screen may be cramped (A5).
- `test-tmp-cleanup`'s `lyx cleanup` must not sweep the hub notice log.

## Acceptance criteria

- With the orch and two runs in flight, the prime's strand window shows the `runs` strand directly above the orch, showing two lines, one per run, with slug, state, loom stage, stage time, run age and session name.
- No runs in flight: the pane shows `no runs in flight` in one line.
- A run starts or lands: the pane grows or shrinks by one line without the operator acting, and a zoomed orch or zoomed Selvage stays zoomed.
- More runs than fit: the last line names how many runs are not shown, and the pane never exceeds the D3 height.
- A non-`done` batten record whose pair is gone does not show; a failed run with its pair present does.
- A run in Run-Shed whose child status file does not exist shows batten's producer, never `status unreadable`.
- A status file unreadable on one poll keeps its previous line; unreadable on two shows `status unreadable`; the other lines still render.
- The overview in a pane whose height disagrees with its reported count still shows a correct summary line.
- `lyx orch start` replaces a live `runs` strand from another build and keeps one from its own.
- VIEW shows the `runs` strand and the orch; the `orch` strand button zooms the orch alone.
- The content-rows operation refuses an unknown strand, a strand with no content declaration, a pane id other than the strand's, and a count below 1; an unchanged count re-applies no layout; a refused first report succeeds at the next poll once the strand is recorded.
- render: a window of positional strands only lays out byte-identically to today; content, sharing and top-anchored strands take the D3 heights, including a top pane's title row and a window too short for every minimum.
- Two seat-table agents get equal heights and the driver collapses; a third equalizes all three; once none is live the bottom-most positional strand expands again.
- The checklist shows every recipe row in order with the right box, the summed time and the visit count for a row reviewed twice, on both child drivers; with more rows than fit, the state line and the running row stay visible.
- A batten notice appends exactly one line with time, text and attach command, also when its delivery took retries; two runs' simultaneous notices give two whole lines; an unwritable log logs a warning and the notice still goes.
- Alt+Down from the bottom strand lands on Selvage, Alt+Down from Selvage wraps to the top strand, both keeping the zoom; Alt+z on Selvage zooms and unzooms it; neither key acts on a hidden batten window's pane.
- The run window's shell line, for both the Konsole and Windows Terminal argument builders: exit 0 ends; non-zero prints the status, then ends when the pair directory is removed within the wait; non-zero with the directory present keeps a shell.

## Notes for the plan writer

- reed's layout rule is described in `internal/reedengine/doc.go` ("The layout rule"); the policy lives in `internal/reedengine/render` (`policy.go`, `height.go`); the hook pins' row-0 title adjustment is `pinsForContent`.
- The orch strand is added in `internal/orchcli`; the `loom-status` strand's keep/replace logic is in `internal/loomcli/bootstrap.go` and `start.go`, with its build sidecar in `statusbuild.go`.
- The bindings and their tmux conditions are in `internal/reedengine/bindings.go`; the window line is `attachLine` in `internal/termwindow/termwindow.go`.
- Notices go out through `deps.Notify` in `internal/battenshed/rownotice.go` and are rendered in `notice.go`.
- `extra-agent-strand` will need a size declaration from D3 when it is built, or its strand becomes the expanding bottom-most one.
