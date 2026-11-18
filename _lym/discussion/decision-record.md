# Decision record — wxb-plan-without

## Goal

While a beam's pawl run lives, the beam's session shows the run's checklist in a window of its own, named for the run's recipe, beside the beam overview.
Today the runs strand takes the overview window from the checklist and the checklist is gone, not moved.
The beam overview also carries one line with the run's current row, naming the checklist's window.

## Scope

In:

- `ensureRunsStrand` in `internal/orchcli/runs.go` gives the overview window to the runs strand and adds the loom status strand back in its own window, in the same call.
- `ensureStatusStrand` in `internal/loomcli/sharedbootstrap.go` adds the status strand through reed's `AddStrandUnless`, so a status strand added concurrently is kept.
- One builder of the loom status strand's add spec, shared by the loom bootstrap and `ensureRunsStrand`, which names the strand's own window for the run's recipe.
- One line for the worktree's own loom run in `lym batten overview`.

Out:

- Reordering `lym beam start` or the pawl recipe so the runs strand comes up before the loom start.
- Removing the status strand when the pawl run ends; no path removes it today, and this task adds none.
- Restoring a checklist strand the operator closed by hand; a later `lym loom start` or resume adds it back, as today.
- Any reed change.

## Decisions

### D1 — The runs strand moves the checklist instead of replacing it

When `ensureRunsStrand` finds no runs strand but a live or dead loom status strand, it keeps its current step:
`ReplaceStrand` on the status strand's guid with the runs spec, so the runs strand takes the overview window, and it writes the runs sidecar as today.
Then, when the worktree's own run, `shedrun.SelfRunID`, has a status whose state is not `shedengine.StateDone`,
it adds the loom status strand again with D2's builder asked for the beside-runs placement, in a window of its own that keeps the viewer.
It writes the loom status strand's sidecar with the new guid and the current build, so the next `lym loom start` keeps that strand instead of replacing it.
The seed, the status and the sidecar are those of the session's own worktree, the location `ensureRunsStrand`'s orch already resolves for the beam, never the caller's.
Every `orchcli.BeamUp` caller passes the beam's own location, and `wireCLI` in `internal/orchcli/cli.go` sets the orch's location and its reed geometry from it.
The re-add goes through reed's `AddStrandUnless` with the loom status role as its unless-name, an operation orch's strand seam gains
(`AddStrandUnless` and `strandNameLocked` in `internal/reedengine/strand.go`).
A skipped add counts as done, since a concurrent `lym loom start` added the strand first, and writes no sidecar.
The loom bootstrap's add of the status strand in `ensureStatusStrand` goes through `AddStrandUnless` on the same unless-name,
and a skipped add counts as kept and writes no sidecar, so neither side of a race between the two adds fails.
The re-add follows a successful replace whatever the runs sidecar write returns, and a failed runs sidecar write's warning is joined with the re-add's own.
A failure of the re-add or its sidecar write is a returned warning naming `lym loom start` as the way to bring the checklist back,
and leaves the runs strand in place.
On this branch a failed `ReplaceStrand` returns a warning naming `lym loom start` as well as `lym beam start`,
since `ReplaceStrand` persists the old strand's removal even when its add fails, so the status strand may already be gone.

Rationale: the displacement happens in this one call, and only this call knows the checklist was taken out.
It covers both orders the pawl run and the beam bring-up can take,
since `statusStrandAddSpec` already places the checklist in its own window when the runs strand came first.
A done run's leftover status strand is not brought back, matching D3, which shows no line for it.

Bound: it adds only the loom status strand, by its fixed role, only right after it took that strand's window, and only for a run not done.

### D2 — One builder of the loom status strand's spec, naming its window for the recipe

The loom status strand's pane command and add spec are built by one function, used by the loom bootstrap's `ensureStatusStrand` and by D1.
The sidecar path the bootstrap reads and D1 writes is declared in the same place.
The builder lives in a package both `internal/loomcli` and `internal/orchcli` already import, never in `internal/loomcli`, which imports `internal/orchcli`.
The builder takes the placement as an input, overview or beside-runs, never a strand list:
the bootstrap picks beside-runs when the session holds the runs strand, as `statusStrandAddSpec` does today, and D1 always asks for beside-runs.
The overview placement gives the overview anchor, as today.
Beside-runs gives the own-window anchor with `KeepViewer`, and a `Display.Window` group named for the run's recipe, read from the run's seed;
the group holds this strand alone.
When the recipe cannot be read, it leaves the group empty, and reed names the window by the strand's role, `loom-status`, as today.

Rationale: two copies of the spec would drift, and D1 runs right after it changed the strands, so a list read before would pick the wrong placement.
A recipe read from the seed hard-codes no name, per `PATTERN-no-hard-coded-choice`.

### D3 — The beam overview shows the worktree's own loom run

`lym batten overview` reads the seed and status of its worktree's own run, `shedrun.SelfRunID`, through `shedrun.ReadSeed` and the status file reader the overview already polls with.
When the seed and status read and the state is not `shedengine.StateDone`, it renders one line with the recipe, the current row as `overviewStage` labels it, and the state.
Every state but done shows, `blocked`, `failed` and `paused` included, since a stuck pawl is the case the operator advances by hand.
The line names the window the loom status strand holds, derived from the session's reed state by the rule reed names it with at launch.
The overview reads `reed.json` under the worktree's anchor `.lym` directory through `reedengine.LoadState`
and finds the strand whose name's role is `loom-status`.
It names the window by that strand's `Display.Window` group when the state's `WindowGroups` record holds the group as shared,
else by the role `loom-status` (`createOwnWindowPaneLocked` in `internal/reedengine/spawn.go`,
`planOwnWindowPlacement` in `internal/reedengine/ownwindow.go`).
When that read fails or no loom status strand is tracked, the line carries no window name.
With no seed, an unreadable seed or status, or a done run, no line is rendered.
An unreadable file is never an error of the overview.

Rationale: the operator sees where pawl stands from the beam overview and is pointed at the checklist.
The strand's group and the group's shared record are what reed named the window by,
so the name stays right when `window_share_max` is 0, where the record holds the group as not shared and the window bears the role.
It also stays right when the seed was unreadable at add time or the strand predates this change, since the strand then has no group.

### Added after Discussion (parent, 2026-11-18): Ignore commit 7c23591 (F5)

Decision: Design against the pair base f98a1ba0 as it stands; do not align with or reference commit 7c23591.

Rationale: This run is a bench arm; both arms must work from the same base so they compare like for like.

## Constraints

- `ensureRunsStrand` never fails `lym beam start`: every failure is a returned warning, as in `internal/orchcli/runs.go` today.
- Reed refuses a second overview strand and refuses a `Display.Window` group on an overview strand,
  and its `AddStrandUnless` skips an add, with no error, while a strand holding the unless-name exists (`internal/reedengine/strand.go`);
  D1 and D2 rely on all three.
- `ReplaceStrand` (`internal/reedengine/replace.go`) removes the old strand and adds the new spec, whatever its anchor.
- A size-1 window group gets a shared window named by the group only when `window_share_max` is at least 1, the default 2 in `internal/reedengine/template_posix.yaml`;
  at 0 reed places the strand apart, in a window named by its role (`planOwnWindowPlacement` in `internal/reedengine/ownwindow.go`).
- An own-window strand's new window opens detached and is selected only when it takes the viewer, which `KeepViewer` never does, on the shared and the apart path alike (`internal/reedengine/spawn.go`, `newAgentWindowSelected`).
- The finished test matches the existing one: a run is finished exactly when its state is `shedengine.StateDone`, as `runStatusUnfinished` in `internal/fabriccli/anchordeps.go` reads it.
- The shared builder and sidecar path are each declared once, per `PATTERN-named-constants`, with no import cycle between `internal/loomcli` and `internal/orchcli`.
- The overview reads files only and never refuses on an unreadable file.
  It takes reed's shared read lock on `reed.json.lock` for its one reed state read, through `ReadJSON` in `internal/state/state.go`,
  and no other lock.

## Auto-mode assumptions

- The checklist's window is named by the recipe alone, so a pawl run's window is `pawl`; the board entry asks for a window "named for pawl", and no other label was asked for.
- The overview line is built, though the entry says "may": it is the only way the operator learns where the checklist went without hunting for the window.

## Open risks

- Parent commit `7c23591` ("Show a beam's pawl checklist in a second overview window", listed by `lym loom peers`, not an ancestor of this worktree's HEAD) fixes the same bug by letting reed hold several overview strands.
  This record designs against the worktree's base, where reed refuses a second overview strand; the run's parent decides whether this bench arm ignores that commit or aligns with it.
- A hand `lym loom start` racing `lym beam start` with neither strand present can still have its overview add refused once the runs strand lands first; that race predates this task and D1 does not widen it.

## Acceptance criteria

- D1: with a loom status strand, no runs strand and a self run not done, `ensureRunsStrand` leaves the runs strand in the overview window and a loom status strand in its own window with `KeepViewer` and the recipe's window group, and the status sidecar records the new strand's guid.
- D1: with the self run done or absent, `ensureRunsStrand` replaces the loom status strand with the runs strand and adds no status strand.
- D1: when the re-add or its status sidecar write fails, `ensureRunsStrand` returns a warning naming `lym loom start`, and the runs strand and its sidecar stay.
- D1: when the runs sidecar write fails, `ensureRunsStrand` still leaves a loom status strand in its own window.
- D1: when the replace of the loom status strand fails, `ensureRunsStrand` returns a warning naming `lym loom start`.
- D1: when a strand holding the loom status role already exists, `ensureRunsStrand` adds none and returns no warning.
- D1: when the loom status role is taken between the bootstrap's strand read and its add, `lym loom start`'s status-strand step succeeds and adds no second strand.
- D1: with a current runs strand, a stale one, or no strand at all, `ensureRunsStrand` behaves as before and adds no status strand.
- D2: asked for beside-runs, the builder gives the own-window anchor, `KeepViewer` and a window group named for the seed's recipe; asked for overview, the overview anchor and no group.
- D2: with no readable seed, the beside-runs spec carries no window group.
- D2: with a runs strand present, `lym loom start` adds the status strand with the own-window anchor, `KeepViewer` and the recipe's window group.
- D3: with a self run in state running, paused, blocked or failed, the overview frame holds one line with the recipe, the row's stage, the state and the loom status strand's window name.
- D3: with the loom status strand in a recipe group that reed's state records as shared, the line names the group;
  with the group recorded as not shared, as at `window_share_max` 0, it names `loom-status`.
- D3: with no tracked loom status strand, the line carries no window name.
- D3: with no seed, an unreadable seed or status, or a done run, the overview frame holds no such line and renders the rest as before.

## Notes for the plan writer

- `internal/orchcli/start_test.go` already drives `ensureRunsStrand` with fake strand ops, a loom status strand among its cases, and `internal/loomcli/bootstrap_test.go` covers `statusStrandAddSpec`; extending both fits tier 1.
- `internal/statuspane` is imported by both callers and owns the strand sidecar helpers, a candidate home for D2's builder.
- `DobbyRunCells` in `internal/battencli/overview.go` shows the seed-plus-status polling D3 can follow.
