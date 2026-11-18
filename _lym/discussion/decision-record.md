## Goal

While a beam's pawl run lives, the beam's reed session shows that run's gate checklist in a window of its own, named for the run's recipe, beside the beam overview.
Today `internal/orchcli#orchCLI.ensureRunsStrand` replaces the `loom-status` strand a pawl run's `lym loom start` put in the overview window,
so the checklist is gone until a later `lym loom start` adds it back.
The beam overview also carries one line naming the worktree's own run, its current row and the checklist's window.

## Scope

In: D1 to D4, and the package docs and `docs/overview.md` lines that state the overview-window rule.

Out:

- A normal task run's session: its checklist keeps the overview window, named as today.
- Reordering `lym beam start` or the pawl recipe so the runs strand comes up before the pawl run's `lym loom start`.
- Removing the checklist strand when a run ends.
- A session where an older build already dropped the checklist strand, leaving none: the next `lym loom start` in the beam's worktree restores it.

## Decisions

### D1 — One checklist strand spec, shared by loom and orch

The checklist strand's pane command, its beside-the-overview spec and its status sidecar path move to `internal/loomengine`, beside `loomengine.LoomStatusStrandName`.
The beside spec builder takes the recipe name and returns an own-window, `KeepViewer` spec with that name as `Title`.
Both callers read the recipe with `shedrun.ReadSeed` on the worktree's `shedrun.SelfRunID`, with no default: a missing or unreadable seed gives an empty title.
`internal/loomcli#statusStrandAddSpec` keeps its placement rule (overview window unless the session holds the runs strand) and builds the beside case from the shared builder.
`internal/orchcli#orchCLI.ensureRunsStrand` builds its moves from the same builder.

Rationale: `internal/loomcli` imports `internal/orchcli`, so orchcli cannot call loomcli's builders,
and `reedengine.StrandStatus` carries neither a strand's command nor its display, so orchcli cannot copy the live strand's spec.
loomcli's own recipe field defaults to tapestry before a seed exists, and a strand kept by its sidecar is never re-specced, so a wrong title would stick.

### D2 — The runs strand moves the checklist out instead of replacing it

`ensureRunsStrand` treats a `loom-status` strand this way:

1. With no runs strand: it replaces the `loom-status` strand with the D1 beside spec, so the checklist leaves the overview window if it held it, then adds the runs strand in the overview window.
   A `loom-status` strand already in its own window is relaunched the same way, since `reedengine.StrandStatus` does not tell the two apart.
2. With a runs strand and a `loom-status` strand that is not live, as a reed server restart leaves both: it replaces the `loom-status` strand with the beside spec beside its runs-strand work.
3. With a runs strand and a live `loom-status` strand: it leaves the checklist untouched.

After each replace of `loom-status` it writes the loom status sidecar with the new strand's GUID and the current build, so the next `lym loom start` keeps the strand.

Failures:

- A failed replace of `loom-status` warns, naming the cause and `lym loom start` in the beam's worktree as the way forward.
  It then reads the strands again: a `loom-status` strand still present is replaced by the runs strand, as today; with none present the runs strand is added.
  Either way the beam overview still comes up.
- A failed add of the runs strand after a successful move warns, naming `lym beam start`; the checklist stays in its own window.
  A concurrent `lym loom start` that put `loom-status` back in the overview window between the move and the add ends here, and `lym beam start` then moves it again.
- A failed sidecar write only warns; the next `lym loom start` then replaces the strand once.

D2 selects no window itself; reed's rules for a viewed window that ends decide where the viewer lands.
A viewer on a window the checklist leaves moves to the newest live agent window;
with none live, it moves to the beam overview window when the runs strand already holds it (case 2), and otherwise nowhere, leaving the choice to tmux (case 1).
The runs strand's overview window, added after a move, is never selected.
The fall-back replace after a failed move selects it when that replace ends the viewed window, as today.
A viewer on any other window stays where it is.

Bound: it acts only on the strand named `loom-status` and the runs strand; every other strand is untouched.
It never fails `start`: every failure is a warning on the Beam-Up report, as today.

Rationale: the replace in `ensureRunsStrand` is the one place the checklist is lost, and its Beam-Up caller runs inside the live pawl run.
Case 2 covers the reed restart, which `lym beam start` otherwise brings back without the checklist.

### D3 — Reed names an own window by its title

`Display.Title` also names the window of an own-window strand that reed places in a window of its own, on every path that names such a window:
the new window of an ungrouped strand, the window of a grouped strand placed apart because its group does not share a window or its shared window is full, and a break-out from the shell window.
An empty `Title`, or one equal to a window name reed reserves (the shell window's and the overview's default), keeps today's name, the strand's role.
A strand that opens or joins its group's shared window ignores `Title`, and the shared window keeps its group's name.
A `Title` two windows share only gives two status-bar buttons of one name:
tmux allows windows of one name, reed addresses an own window by its window id and finds the shell window by its console pane,
and `OpenWindow`'s same-name lookup skips windows carrying `@lym_own` (`internal/reedengine/doc.go`, window-order and detached-window paragraphs).

Rationale: the operator asked for a window named for pawl, and the role `loom-status` is pinned for lookups.
The name comes from the run's seed, not a literal.

### D4 — The beam overview names the worktree's own run

The runs strand's frame opens with one line for its worktree's own run while that run has a seed and a status file that is not done:
the recipe, the current row, the run's state, and the checklist window's name, which is the recipe.
With no seed, no status file or a done run, no line is drawn.
The line reads the seed through `shedrun.ReadSeed` and the status through the existing `statuspane` poll, without locks, like the frame's other lines.

Rationale: the operator can see where pawl stands from the overview and find its window by name.

### Added after Discussion (parent, 2026-11-18): Ignore commit 7c23591 (F2)

Decision: Design against the pair base f98a1ba0 as it stands; do not align with or reference commit 7c23591.

Rationale: This run is a bench arm; both arms must work from the same base so they compare like for like.

### Added after Discussion (arbiter, 2026-11-18): Redesign from this arm's base without reconciling with 7c23591

Decision: This record designs the fix from this worktree's base f98a1ba0 and does not reconcile with parent commit 7c23591 ("Show a beam's pawl checklist in a second overview window"); D1 to D4 stand as written. The Open risks bullet on 7c23591 is answered by this decision and is dropped from the record; Plan and implementation read no code from 7c23591.

Rationale: The board entry marks this task a bench arm of plan-writer-explorer-forks, never to be released. That beam's entry runs each arm from the reference run's frozen inputs and the base commit it started from, so reconciling with a later commit would change the bench's inputs. Settling this narrows nothing and widens no scope.

## Constraints

- `PATTERN-shed-driver`: the seed and status file paths come from `internal/shedrun` only.
- `PATTERN-durable-vs-ephemeral-state`: the sidecar stays under `.lym`, at the path `loomengine.LoomScratchDir` gives today.
- The window name is read from the run's seed; placement is keyed on the runs strand's presence, never on a recipe name.
- `PATTERN-test-speed`: reed's window naming needs the tmux tier; the other criteria are pure or fake-seam tests.

## Auto-mode assumptions

- The window is named by the bare recipe name, so a pawl run's window reads `pawl`.
- D4's overview line is in scope; the entry offers it as optional.

## Open risks

- Parent commit `7c23591` ("Show a beam's pawl checklist in a second overview window") fixes this bug with another design: several overview strands through a display order, and no replace in `ensureRunsStrand`.
  It is not in this worktree's base, and both rewrite `statusStrandAddSpec` and `ensureRunsStrand`.
  This entry is a bench arm marked never to be released; question for the parent: is the record meant to redesign the bug from this base without reconciling with `7c23591`?

## Acceptance criteria

- With a `loom-status` strand and no runs strand, `ensureRunsStrand` leaves the runs strand in the overview window and the `loom-status` strand in a window of its own with `KeepViewer` and the seed's recipe as `Title` (D1, D2).
- After that move the loom status sidecar names the new `loom-status` GUID and the current build, so `resolveStatusStrandAction` answers keep (D2).
- With a runs strand and a dead `loom-status` strand, `ensureRunsStrand` relaunches `loom-status` with the beside spec; with a live one it leaves it untouched (D2).
- When the move's replace fails before the old strand is removed, `ensureRunsStrand` warns naming `lym loom start` and replaces `loom-status` by the runs strand; when it fails after the removal, it warns and adds the runs strand (D2).
- When the runs strand's add fails after a successful move, `ensureRunsStrand` warns naming `lym beam start` and `loom-status` stays in its own window (D2).
- With an unreadable or missing seed, both loomcli's and orchcli's beside spec have an empty `Title` (D1, D3).
- `statusStrandAddSpec` with a runs strand present returns the shared beside spec for the seed's recipe; without one it returns the overview spec with no `Title` (D1).
- Reed opens an ungrouped own-window strand, and a grouped one placed apart in a window of its own, with a non-empty `Title` in a window of that name; one with an empty or reserved `Title` in a window named by its role; and a strand that opens or joins its group's shared window in a window named by its group, ignoring `Title` (D3).
- A strand broken out of the shell window takes the same name rule (D3).
- The runs strand's frame opens with the own-run line naming recipe, row, state and window name while the worktree's run has a seed and a status file that is not done, and draws no such line with no seed, no status file or a done run (D4).
- Review: the `internal/orchcli` and `internal/loomcli` package docs, the `Display.Title` comment and `docs/overview.md` state that the checklist moves to its own window, named for its recipe, when the runs strand takes the overview window (D2, D3).
