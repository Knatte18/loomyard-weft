# Decision record — beam-overview-window

## Goal

The overview window becomes the beam's own window: it is named `beam: <slug>`, `beam: prime` in the prime, and draws one dobby section and one orch section, so each run shows once, in the dobby section.

## Scope

- In: the overview window's tmux name, set by the runs strand the orch ensures in the prime and in each beam.
- In: the frame `lyx batten overview` draws: a dobby section, then an orch section; the separate list of runs in flight goes.
- In: the dobby section's run lines carry the stage detail the run list carried, and `lyx beam status` prints the same lines.
- In: the orch section of a beam's window shows that beam's orch and the orchs of every beam below it; the prime's shows every orch.
- In: the docs that describe the window: `internal/orchengine/doc.go`, `internal/reedengine/doc.go`, `internal/beamcli/doc.go`, `internal/battencli/cli.go`'s module comment, `docs/overview.md` and the orch role stencil where it names the window.
- Out: the loom status strand of a task pair, which keeps its window named `overview`.
- Out: renaming the `runs` strand role or the hidden `lyx batten overview` verb.
- Out: `lyx beam start` and the other parts of `beam-start-and-ownership`.

## Decisions

### The overview window's name comes from the display

- Decision: `render.Display` gains an optional `Title`, read only for an overview strand; reed names the overview window `Title`, and `overview` when it is empty. The orch's runs strand sets `beam: prime` in the prime and `beam: <slug>` in a beam.
- Rationale: reed cannot tell a prime from a beam, and the loom status strand of a task pair has no beam to name; the display is persisted with the strand, so a relaunch keeps the name.

### The runs strand keeps its role

- Decision: the strand stays named `runs` and the verb stays `lyx batten overview`.
- Rationale: an existing session's strand is found and replaced by that role on the next `lyx orch start` after a build change, so the new window name reaches running sessions without a migration.

### Dobby section first, then the orch section

- Decision: the frame draws the dobby's lines, a blank line, then one orch line per orch shown, in the order the board entry lists the sections. A worktree with no dobby, a task pair, draws the orch section alone.
- Rationale: the board entry's design names the two sections in that order.

### Each run once, in the dobby section, with its stage detail

- Decision: the list of the prime's runs in flight is removed. Each run line of the dobby section reads slug, state, stage and time in stage as the run list drew them (a covering pause as `paused <minutes>`, a run in `Run-Shed` with the child's numbered stage), then the dobby's pass cell. One function renders the cells for the pane and for `lyx beam status`.
- Rationale: almost every run in flight sits in `Run-Shed`, so the dobby's own producer alone loses what the run list showed; age and reed session are dropped to keep the line short.

### Orch section scoped to the beam and the beams below it

- Decision: a window shows the orch of its own worktree and of every beam whose parent chain, read through `hubgeom.ParentOf`, reaches that worktree; the prime's window shows every orch of the hub.
- Rationale: the board entry's design says a beam's window shows its own orch and the orchs of every beam below it.

## Constraints

- `PATTERN-cwd-resolution`: the pane keeps resolving its location through `internal/lyxcwd` at the verb boundary.
- `PATTERN-pair-parent`: a beam's parent is read only through `hubgeom.ParentOf`.
- `PATTERN-daemon-wakeups`: the pane's text still changes at most once a minute when idle, so its poll keeps backing off.
- `PATTERN-test-speed`: tests use injected clocks and run in parallel.

## Auto-mode assumptions

- The window name uses the literal form `beam: <slug>` with a space after the colon; tmux takes it as a window name.
- A beam nested under another beam is placed by its origin record's parent worktree, as `hubgeom.ParentOf` answers.

## Open risks

- A session started by an older build keeps its `overview` window until `lyx orch start` replaces the strand for the new build.
- A window name with a colon and a space could clash with a reed lookup that targets windows by name; reed targets the window by id after creation, which the tests cover.

## Acceptance criteria

- An overview strand with a `Title` opens its window under that name; one without opens it as `overview`.
- The orch's runs strand spec carries `beam: prime` in the prime and `beam: <slug>` in a beam.
- The frame is the dobby's lines, a blank line, then the orch lines; no run list is drawn, and a run shows once.
- A dobby run line in `Run-Shed` shows the child's numbered stage and time in it; a paused run shows `paused <minutes>`; an unreadable status shows `status unreadable`.
- A beam's window lists its own orch and a nested beam's orch, and leaves out the prime's orch and a sibling beam's; the prime's lists every orch.
- `lyx beam status` prints the same dobby lines the pane draws.
