```yaml
lens: board-fit
outcome: reported
```

Board-fit lens: the record's fit with the hub's other board entries, peer runs and parent commits.

**Checked list**

- Decision notes for this task: none. No `lyx board list` title reads `Decision for umbrella-anchor:`, and the record has no `Added after Discussion` entries. No note was out of reach.
- Peer `dobby-overview-window` (T4, running · Darn-Write): overlap, no conflict.
  - Its decisions add a dobby view in `lyx batten overview`, a `lyx beam status` verb and a refusal text in dobby's start record. They also move the run and task listings from `beamcli` to `battencli`.
  - This record touches `beamcli/dobbydeps.go` `pairLocation` and `hubgeom` `DobbyGeometry`. That is the same packages at different places, a text-only overlap.
  - The peer's new reads of a beam's dobby lock and state go through the dobby geometry. This record's D3 makes that geometry, and every other-worktree site, take the worktree's own Location. Whichever run lands second follows D3's rule.
  - Neither run changes the same behaviour or PATTERN entry in a different way. The peer's `PATTERN-cwd-resolution` constraint only restates the told-geometry rule, which the declared change keeps.
- Parent commit `f9f8f2c` ("Dobby counts no done run against max_runs…", `Lyx-Task: dobby-skip-done-runs`): no conflict.
  - It changes only which runs dobby plans and counts.
  - D7's refusals ("its run status names an unfinished run", and for the prime "any live batten run") are unaffected. The commit is the cause of this task's late start, which the support log's Turn 1 already accounts for.
- Entry `fabric-remote-async` (E19): aligned, no conflict.
  - D8's census reads only fetched refs and makes no network call.
  - D7 step 7 spawns the push async.
  - `beam create`'s fork keeps only the branch-exists probe, which the note's one exception allows.
- Entry `beam-promote-keeps-discussion` (E14): no conflict.
  - The note says `lyx beam promote` is unused for now and proposes carrying the Discussion record into a promoted beam.
  - D10 adds only an anchor-mismatch refusal to promote, before any reset. That neither blocks nor duplicates the idea, and the record states promote's marker-change path as the reason it is in scope.
- Entry `loom-beam-darns` (F20): no conflict. It is a future design where a task's worktree becomes a beam, and nothing in this record contradicts it.
- Project `workspaces` (P1): no conflict. It is an umbrella project with no design text this record could contradict.

**Findings**

None. No landed commit, peer run or board entry contradicts or duplicates a decision. Nothing raises a parent question, and there is no ordinary misalignment for the writer to settle.

**Tool calls**

1. `lyx board list`: the lens requires it; my context had no board listing.
2. `lyx loom peers`: the lens requires it; it names the peer and the parent commit.
3. `lyx loom peers show dobby-overview-window --record decision`: the closest peer; its decisions were not in context.
4. `git show --stat f9f8f2c`: the parent commit `lyx loom peers` lists; its content was not in context.
5. `lyx board get` for `beam-promote-keeps-discussion` and `fabric-remote-async` (one call): the touching notes on promote and on remote async work; their bodies were not in context.
