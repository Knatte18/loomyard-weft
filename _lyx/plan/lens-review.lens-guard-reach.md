```yaml
lens: guard-reach
outcome: reported
```

Lens guard-reach: whether each guard the plan adds can be reached, and whether a guarded site recurs unguarded.

**Findings**

1. **BLOCKING — Card 10, D10 checkout: the refusal misses a target weft branch that is only on origin, so the checkout switches across a marker change unguarded.**
   - Card 10 reads the target tip's anchor only when the branch "exists locally or as origin's remote-tracking ref".
   - `switchOrForkWeft` calls `resolveWeftBranch(..., consultOrigin=true)`. Its origin probe (`remoteHeadTip`, `internal/fabricengine/weftbranch.go:39`) finds a branch origin holds even when no tracking ref was ever fetched.
   - It then fetches it and creates a tracking branch (`weftbranch.go:48-56`), and `Checkout` switches to it and rewires at `l`'s anchor (`checkout.go:111`).
   - So a branch another machine pushed and this machine never fetched carries another marker, yet passes the refusal and gets switched to. That is the half-rewired pair D10 forbids.
   - Card 10's test row only builds the target branch locally, so it never reaches this state.

2. **MEDIUM — Card 12, D10 promote: the guard compares working-tree markers, not the tree the reset carries, and is wrong when the parent pair is absent here.**
   - `Anchors` reads the parent with `lyxcwd.NamedWorktree(prime.HubPath, parentWorktree).AnchorRel`.
   - `ResetRecordsToParent` resets to `recordsBranchTip(repo, parentRecords)`, which is the local branch or origin's tracking ref (`destroy.go`, `recordsBranchTip`). Nothing requires the parent's worktree to be present on this machine.
   - With the parent's weft absent, `NamedWorktree` answers D2's fallback (Bolt or `"."`), so the guard can pass a task whose parent tip carries another marker, or refuse a matching one. D3 says no anchored path is used for a pair whose weft is gone.
   - The comparison D10 states ("resetting the records to the parent's tree carries the parent's marker") is `treeAnchor` of the parent's records tip.
   - `TestPromote`'s fake rows cannot reach this case.

3. **MEDIUM — Card 16, D7 step 1: the dobby gate stops passes and tasks, but a dobby's board-tidy agent still spawns while a move record stands.**
   - Card 16 lets `daemon.tick` go "on to its other per-tick work".
   - That includes `tidyStep`, which spawns the board-tidy agent as a shuttle strand (`internal/dobbyengine/tidy.go:1`, `BoardTidyRole`).
   - That strand's state and working directory sit under the worktree's anchor, which D7's rationale says the move needs gone ("every agent and daemon in the worktree has its cwd or state at the anchor").
   - Card 17 checks for strands only before it starts, and its pass wait counts driver locks and run-process children, not a tidy strand spawned after the record is written.
   - So the site D7 guards recurs on an unguarded path, and no test row covers it.

4. **LOW — Card 18, D6 re-run: a re-run of `lyx beam create` without `--anchor` over a present pair with a standing move record skips recovery and writes into the weft mid-move.**
   - `createSteps` calls `EnsureAnchor` only "on a present pair with a non-empty anchor".
   - Otherwise it goes straight to `MarkBeam`, which writes and commits the origin record (`origin.go`, `MarkBeam` → `WriteOrigin` and `CommitRecordsPaths`), and then to `SpawnPush`.
   - For a pair whose record stands, that commits at whatever anchor `NamedWorktree` reads from a half-moved marker. No card refuses `MarkBeam` or create's present-pair path on a standing record.

5. **LOW — Card 8, D2 and D5 adopt: an adopted weft branch whose committed marker is empty is neither refused nor clearly handled.**
   - `committedMarker` returns `""` with `found` true for an empty marker.
   - Card 8's adopt path acts only on "no committed marker", and `treeAnchor` skips an empty committed marker for the tree heuristic.
   - So an adopt over such a branch either uses an empty `AnchorRel` or silently rewrites the marker, with nothing reaching D2's refusal of an empty own marker. No `TestReconcile_MigratesMarkerlessWefts` row reaches it.

6. **LOW — Card 13, D7 record: `lyx fabric checkout` is not refused while a move record stands.**
   - A checkout during a killed move (record standing, lock free) switches the weft branch the record's `A`, `B` and `U` describe.
   - The re-run's "landed" test (`HEAD`'s committed marker is `B`, `U` an ancestor of `HEAD`) then reads another branch's `HEAD`, and its reset of `A/<names>` and `B/<names>` acts on that branch.
   - Card 10's anchor refusal only covers a marker difference: a target branch with the same marker passes, so the guarded site recurs.

**Tool calls**
- `internal/fabricengine/pushanchored.go:60-100` and `resolveWeftBranch`: to see whether checkout's origin adoption and the code push replay; the context lacked both bodies.
- Grep for the dobby `Tick` hook's production setter: to see whether a pass can start outside `drivePasses` and `startReady`; the context lacked it.
- `internal/dobbyengine/loop.go:55-64` and the repo-wide `Tick:` grep: to confirm the hook is unset in production; the context lacked it.
- `internal/dobbyengine/tidy.go` head and `tidyStep`: to see what the tidy step spawns while a record stands; the context lacked it.
- `internal/fabricengine/pushclassified.go:93-160`: to see whether any push path other than `retryWeftPush` replays; the context lacked the push job body.
