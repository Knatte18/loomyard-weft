```yaml
lens: goal-scope
outcome: reported
```

Lens: goal-scope. I checked the record against the `umbrella-anchor` board entry and its decision notes.

**Goal points of the entry, met or not:**

- Every weft branch may carry a `.lyx-anchor` at its own root: met (D1).
- The weft sits at `<hub>/<slug>-weft`, so no anchor index on Bolt is needed: met (D1; the index is listed under Scope Out).
- Inheritance through git, and a worktree with another anchor commits its own marker: met (D5, D6, D7 step 3).
- "No marker stays on Bolt", and `fabric clone --subpath` writes the first marker on `main-weft`: partly met. D4 covers the clone. D8 removes the Bolt marker only once the census passes, and nothing bounds when that happens (finding 2).
- `lyxcwd.resolveCore` reads the worktree's own marker, and every `AnchorRel` reader follows: met (D2, D3).
- The parent keeps its own marker at landing, "kept out of the merge as raddle is": met by a stated fact (Constraints: the weft never merges).
- `beam create --anchor <dir>`, defaulting to the spawning worktree's anchor: met (D6, D5).
- Where the work is (the `lyxcwd` and `codebinding` reads, the clone, reconcile and unwire writers, `mergepaths`, `warplayout`, `pairteardown`, `forkrecords`, `_portals`, `_launchers`, pathspecs, workspace files): met (D1–D5, D8, D3, D7 steps 5 and 6, Constraints).
- The anchor-change verb removes junctions, moves the weft files including `<anchor>/.lyx`, commits the marker, rewires and regenerates the workspace files: met (D7).
- "No run may be in flight in the worktree while it moves": not met for passes of child runs (finding 1).
- Children keep the old anchor, the messy case: met (D7, "Child worktrees keep their own markers").
- Run as one loom task on profile `solid`: not a record matter.
- Directions the entry rules out (a hub per subdirectory, an index on Bolt): kept out. The D10 refusals are protective consequences of per-worktree markers, not unlicensed scope.

**Decision notes:** `lyx board list` has no note titled `Decision for umbrella-anchor:`, so I read none. The record has no `Added after Discussion` entries.

**Findings:**

1. MEDIUM, class `design`, at D7 (step 1, the refusals list) and the acceptance criterion "while a pass that dobby had in flight finishes".
   - The entry says "No run may be in flight in the worktree while it moves".
   - D7 refuses only the worktree's own runs, and lets passes the dobby already spawned keep running through the move.
   - Those are passes of child runs whose parent is this worktree (`internal/dobbyengine/doc.go`, "The passes" section: each tick lists the runs whose parent worktree is this dobby's).
   - A child pass in Finalize lands into this worktree. Per CLAUDE.md, landing regenerates `_lyx/raddle/` against the parent, and `internal/fabriccli/merge_verbs.go:128` stages `_lyx/raddle/notes.md` in a merge.
   - So a landing can write through the parent's `_lyx` junction while steps 2–4 have it removed or moving.
   - The record does not bound this case. Fix: D7 should either refuse while any child run of the worktree has a pass in flight or holds the parent lock, or state what such a landing can write mid-move and why that is safe.

2. LOW, class `design`, at D8 (the Bolt marker removal).
   - The entry requires that no marker stays on Bolt.
   - D8 keeps the Bolt marker while any counted origin branch is unfetched or unmarked: "a branch whose ref is not fetched keeps the Bolt marker" (`decision-record.md:141`).
   - Nothing says what brings the census to completion, such as which fetch fills the refs, or what happens to a markerless branch another machine pushed but never reconciles. Removal can therefore be put off without limit.
   - Fix: state what drives the census to completion, or what bounds the wait.

**Tool calls:**

- `lyx board list`: to find this task's decision notes; my context had only the entry body.
- `lyx board list` filtered for "decision for" and "anchor" titles: to confirm no decision note exists.
- `grep` for "raddle" in `doc.go` files: to check whether landing writes the parent's raddle; it returned nothing.
- `grep` for production files naming raddle: to locate the landing-merge code.
- `grep` for "raddle" in `merge_verbs.go` and `fabricengine/doc.go`: to confirm that a landing merge stages `_lyx/raddle/` in the parent.
