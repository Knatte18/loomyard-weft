# Decision record — pawl checklist in a second overview window of a beam's session

## Goal

While a beam's pawl run lives, the beam's session shows the run's checklist in an overview window of its own, named for pawl, right after the beam overview window.
Neither strand replaces the other, whichever of `lym loom start` and `lym beam start` adds its strand first.

## Scope

In: D1 to D5, and every package doc and comment in reed, orchcli and loomcli that states one overview strand per session.

Out:

- A line in the beam overview (`lym batten overview`) naming pawl's current row.
- A split of the one overview window into two panes.
- Any change to the order in which `lym beam create`, `lym beam start` and the pawl run's `lym loom start` run.

## Decisions

### D1 — The overview anchor holds several strands, ordered by a display order key

reed's overview display gains an integer order key, read only for an overview strand and persisted with the display in reed.json; its zero value is the first place.
Overview windows sit at the session's lowest indexes, before every agent, shell and hidden window, in ascending order key.
A new, relaunched or replacement overview window opens right after the last overview window whose key is not higher than its own, and before the lowest-index window when there is none; so windows with equal keys sit in the order they last opened.
`AddStrand`'s refusal of a second overview strand and `ReplaceStrand`'s refusal while a third strand holds the overview are removed.
`UpdateStrand` refuses a change to an overview strand's order key, as it refuses an anchor move, naming remove and add as the way forward.

Bound: reed no longer caps the overview strands of a session.
`lym reed add` already refuses the overview anchor as engine-only (`internal/reedcli/add.go`), and shuttle passes its spec's display through but no shuttle caller sets the overview anchor, so the runs strand and the loom status strand are the only overview spawners.
Both add under an explicit name, which `AddStrand` refuses while another strand holds it, so neither adds a second copy of itself.

Rationale: the operator's chosen design; a key whose zero value is the first place leaves every overview display recorded by an older lym at the front.

### D2 — A fallback to the overview window takes the lowest-index overview window

Every focus rule that falls back to the overview window takes the lowest-index one.
`reedengine.relevantWindow`, through `agentWindowPanes`, and so attach, a viewed window ending and `FollowRun`, tells overview panes from reed state's displays and takes the index from the session's window listing.
The engine-less `reedengine.followTargetWindow` behind `lym reed follow` takes the lowest-index window carrying the overview marker.
No overview window is an agent window, so the newest live agent window still wins over every overview window.

Rationale: in a beam's session the lowest-index overview is the beam overview, the window the fallback leads to today.

### D3 — The loom status strand always takes the overview anchor, after the runs strand, named for its recipe

`loomcli.statusStrandAddSpec` always returns an overview display, in every session: its order key one past the runs strand's, and its Title the run's recipe name, so a pawl run's checklist window reads `pawl`.
The branch that gave it an own window with `KeepViewer` when a runs strand was present is removed, so the add spec no longer reads the session's strands.
The order keys are named constants: the runs strand's in orchcli, which loomcli already imports, and the status strand's derived from it.

The recipe name is the `recipe` field `lym loom start` already holds when it ensures the status strand.
It equals the seed's recipe on every start: `resolveRecipe` (`internal/loomcli/arm.go`) reads it through `resolveSeedRecipe`, and a start with no seed writes the seed with that same function's answer in `seedAndCommitBootstrap` (`internal/loomcli/sharedbootstrap.go`) before step 4 ensures the strand.

Rationale: the checklist's place no longer depends on which strand came first, the beam overview stays first, and the title comes from the run's own record, never from a branch on a recipe name (`PATTERN-no-hard-coded-choice`, `PATTERN-loom-contract`).

### D4 — The runs strand is added beside a loom status strand, never in its place

`ensureRunsStrand` stops looking for a loom status strand: with no runs strand it adds one, with order key zero; a current runs strand is kept and a stale one replaced, as today.

Rationale: reed admits both strands (D1), so the replacement that removed the checklist is no longer needed.

### D5 — An existing beam session is put right by the pawl run's next loom start, with no migration step

A session from before this change holds the status strand in an own window, or holds none because the runs strand replaced it.
The pawl run's next `lym loom start` finds its sidecar naming another build, or finds no status strand, and replaces or adds it under D3, which moves it into the second overview window.
That start comes from a batten pass that spawns the run's loom session, or from the operator: `lym loom start` in the beam's anchor is re-entrant and keeps a live driver.
An overview display with no order key, written by an older lym, reads as key zero.

Bound: until that start, such a session keeps the old placement, and an old status strand counts as an agent window for the focus rules.
An old-key status strand ties with the runs strand at key zero, so the two sit in the order they last opened, on add, replace and relaunch alike.
An older lym process still doing reed ops refuses a second overview add of its own, and one that rewrites reed.json drops the order key, which then reads as zero with the same tie.

Rationale: the status strand's own keep, replace or add already runs at every loom start; a separate migration adds a code path for a state that heals itself.

## Constraints

- reed learns no beam, loom, pawl or recipe name; the order key and the title are told by each spawner.
- An overview window keeps every property it has today besides the cap: its own window, its Title as the window name, no group, no `@strand_color`, no `@lym_unseen`, never selected when it opens, and a replacement that ends the viewed overview window selects its new window.
- After the overview windows, the window order stays: agent windows in start order, the shell window, then hidden windows.
- `PATTERN-named-constants`: each order key is one named constant, declared once.
- `PATTERN-test-speed`: each scenario at the lowest tier that fits; placement and focus choices are pure functions over window listings and are covered without tmux where they can be.

## Auto-mode assumptions

- The pawl checklist's window is named with the recipe name alone, `pawl`, not `pawl: <slug>`: the beam overview's window already names the slug, and a short name keeps status line 1 narrow on a phone.
- A task run's checklist window, today named `overview`, is named with its recipe name too, such as `tapestry` or `darn`; D3 applies the rule to every loom run instead of branching on whether the worktree is a beam.

## Open risks

None.

## Acceptance criteria

- D1: a session holding an overview strand with key 1 accepts the add of a second overview strand with key 0, and its window opens before the first; added in the other order, the windows end in the same order.
- D1: with two overview windows, an agent window opens after both, and the shell and hidden windows stay after the agent windows.
- D1: replacing or relaunching either of two overview strands with distinct keys keeps the windows' order, and a replacement that ends the viewed overview window selects its new window.
- D1: two overview strands with equal keys sit in the order they last opened.
- D1: an update that changes an overview strand's order key is refused, naming remove and add.
- D2: with two overview windows and no live agent, the relevant window and the follow target are the lower-index overview window; with a live agent, they are the agent's window.
- D3: the status strand's add spec is an overview display with the order key after the runs strand's and the run's recipe name as its Title, whether or not the session holds a runs strand.
- D4: `lym beam start` in a session whose loom status strand holds an overview window adds the runs strand beside it and leaves the status strand, its pane and its window untouched.
- D5: a status strand recorded in an own window is replaced into an overview window at the next loom start whose sidecar names another build; an overview display recorded with no order key loads as key zero.

## Notes for the plan writer

- `reedengine.overviewWindowInsertArgs` places by the lowest-index window today; D1's placement needs each listed window's order key, from reed state's displays or from a window option set where the overview marker is set.
- reed's package doc carries the singular rule in its "Overview anchor" and focus-rule paragraphs.
