```yaml
lens: goal-scope
outcome: reported
```

**Goal points of the entry, and whether the record meets them**

- A `.lyx-anchor` at each weft branch's root, set per worktree: met (D1).
- lyx finds the marker from `<hub>/<slug>-weft`, no anchor index on Bolt: met (D1; Bolt index is Out).
- Inheritance from git; a worktree with another anchor commits its own: met (D5, D6, D7 step 4).
- No marker stays on Bolt; `fabric clone --subpath` writes the first marker on `main-weft`: met (D4; D8 removes the Bolt marker after the census).
- `lyxcwd.resolveCore` reads the worktree's own marker, and every `AnchorRel` reader follows: met (D2, D3 with the constructor and scan).
- At landing the parent keeps its own marker, "kept out of the merge as raddle is": met by fact (Constraints: the weft is never a merge participant, so no mechanism is needed); this also answers the entry's `mergepaths` hint.
- `lyx beam create --anchor <dir>`; default is the spawning worktree's anchor: met (D6, D5).
- Where the work is — `lyxcwd` and `codebinding.go`'s read: met (D1, D2, D4's `probeWeftBinding`); writers clone, reconcile, unwire: met (D4, D8; D8 decides unwire leaves the marker, a stated choice); other-worktree path builders `warplayout.go`, `pairteardown`, `forkrecords.go`: met (D3, D5).
- One-anchor keys of `_portals/`, `_launchers/`, weft pathspecs, hub and beam workspace files: met (D3 sweep rule; D7 step 6 and auto-mode assumption; D3 pathspecs; D7 step 7 and acceptance).
- Changing an anchor: remove junctions, move weft files including the warp's machine-local `<anchor>/.lyx/`, commit the marker, rewire, regenerate workspace files: met (D7 steps 3–7).
- "No run may be in flight in the worktree while it moves": met at the goal level (D7's refusals of the worktree's own run, strands and driver lock, and its wait on child passes); whether the bounds hold is for the bound-breaker and interactions lenses, not this one.
- Children keep the old anchor: met (D7 "Child worktrees keep their own markers and anchors").
- Run section (one loom task, profile `solid`, after `adopt-runs-tests`): not record content; the support log confirms `adopt-runs-tests` landed (`42dee71ec`).

**Scope and licence checks**

- D10's checkout/pull/push/promote refusals and the cross-machine integration are licensed consequences of the in-scope change verb and per-worktree anchors (without them a marker change on one machine would rewire another machine's pair unguarded); no other board entry claims this work, and the board list shows no anchor-related entry besides this one.
- No deferral to Plan or a later task is in the record; the Open risks' operator steps are operator procedure, not deferred research.
- No direction the entry rules out: a separate hub per subdirectory and a Bolt anchor index are both Out.

**Decision notes**

- The board list shows no note titled `Decision for umbrella-anchor:`; none read, none unreachable.
- The record has no `Added after Discussion` entries.

**Findings**

None.

**Tool calls**

- `lyx board list` (filtered for anchor/Decision-for titles): my context held the entry but not the board list, needed to find decision notes and overlapping entries.
