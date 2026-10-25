```yaml
lens: goal-scope
outcome: reported
```

Goal-scope lens: I checked the decision record against the `umbrella-anchor` board entry and its decision notes.

**Decision notes:** The board list in my context has no note titled `Decision for umbrella-anchor:`. I had none to read, and none was out of reach.

**Goal points of the entry, and whether the record meets them:**
- `.lyx-anchor` at each weft branch root, set per beam and in effect per worktree — met (D1).
- lyx finds the marker from the `<hub>/<slug>-weft` path, with no anchor index on Bolt — met (D1; the index is under Out).
- Inheritance through git forks, and a worktree with another anchor commits its own marker — met (D5, D6, D7).
- No marker stays on Bolt; `fabric clone --subpath` writes the first marker on `main-weft`; `resolveCore` reads the worktree's own marker — met in design (D2, D4, D8), but see finding 1.
- The parent keeps its marker when a child lands — met as a cited fact: the weft never merges (Constraints).
- `beam create --anchor`, defaulting to the parent's anchor — met (D6).
- The marker readers `lyxcwd` and `codebinding` — met (D2, D4).
- The marker writers `clone`, `reconcile`, `unwire` and `mergepaths` — met (D4, D8). For `mergepaths`, no mechanism is needed, per the Constraints fact.
- `warplayout`, `pairteardown` and `forkrecords` resolve the other worktree's own anchor — met (D3, D5).
- The one-anchor keys of `_portals/`, `_launchers/`, weft pathspecs and the workspace files — `_launchers` is met (D7 step 5), `_portals` is met by assumption (keyed by the prime's anchor), and the workspace files are met (D3, D7 step 6). Weft pathspecs are only implicit; see finding 2.
- The change verb removes junctions, moves the weft files including `.lyx`, commits the marker, recreates the junctions and regenerates the workspace files — met (D7).
- No run in flight while the files move — met (D7 refusals).
- A worktree with children (the messy case) — met: the children keep their own anchors (D7).
- One `solid` loom task, run after `adopt-runs-tests` — met; that task has landed (support log turn 1).

**Unlicensed work or deferrals:** None that is blocking.
- D9 (exclude census), D10 (checkout and promote refusals, pull rewire) and the print mode of `lyx fabric anchor` are direct consequences of per-worktree anchors, so they are licensed.
- Leaving `.vscode/` out of scope does not drop a goal point: the entry asks to regenerate the workspace files, not the IDE config.

**Findings:**

1. **MEDIUM — D8 (Bolt marker removal).** The origin census can block forever, so "no marker stays on Bolt" may never be reached.
   - Evidence: D8 removes the Bolt marker only when every weft branch on origin carries a marker. Reconcile only migrates weft worktrees present on the machine.
   - A weft branch on origin with no live pair on any machine never gets a marker. Examples: an orphaned branch waiting for `lyx fabric cleanup`, or a branch abandoned on another machine.
   - The entry states that no marker stays on Bolt as an end state, and nothing in the record guarantees it is reached.
   - Fix: either the census counts only branches of live or owned pairs, or reconcile commits a marker onto origin-only branches. In both cases, state the bound.

2. **LOW — Scope / D3 (weft pathspecs).** The entry names "weft pathspecs" as a one-anchor key, but no decision states that every weft pathspec for a worktree is scoped under that worktree's own anchor.
   - Evidence: `ScopedPathspec(l.AnchorRel, …)` sites such as `remove.go`, `fabriccli/weft_verbs.go` and `destroy.go`'s reset take the acting Location.
   - D3's general rule implies the fix, but the entry's item has no explicit answer.
   - Fix: add one sentence to D3 tying weft pathspecs to the target worktree's own anchor.

Tool calls: none. The board entry and board list were already in my context from the writer's Step 1, so I made no further reads.
