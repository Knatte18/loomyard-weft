```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens review of the diff. I found four LOW `design` findings and no higher severity. Four files were only partly or never read, and I have no tool calls left to check them (see the last line).

**Per function or branch: case run, correct or not**
- `lyxcwd.readRecordedAnchor`, own marker valid: returns the anchor. Correct.
- `readRecordedAnchor`, own marker invalid: error naming the file. Correct.
- `readRecordedAnchor`, own marker empty or absent: falls to the Bolt marker, then the stale-spelling guard, then `"."`. Correct.
- `readRecordedAnchor`, own marker present but unreadable (a directory, or EACCES): see finding 3.
- `lyxcwd.NamedWorktree`, any name with no weft sibling: `"."` or the Bolt value, with no existence check. Correct.
- `hubgeom.DobbyGeometry`, prime: the prime's move record only. Correct.
- `DobbyGeometry`, beam: the beam's own record first, then the prime's, at the beam's own anchor. Correct.
- `DobbyGeometry`, beam with an invalid marker: returns an error naming the worktree. Correct.
- `DobbyStart.MoveInProgress`, `moveGateSlugs`: the beam and the prime, and the prime once for the prime. Correct.
- `fabricengine.MoveInProgress`, no record: not standing. Correct.
- `MoveInProgress`, record with a free lock: "interrupted" with the re-run naming the record's `B`. Correct.
- `MoveInProgress`, record with a held lock: see finding 2.
- `heldMoveRefusal`, record naming a live process versus any other holder: gives "in progress" versus "starting", as D7 requires. Correct.
- `BeginAnchorChange`, empty own marker: refused first, before the lock. Correct.
- `BeginAnchorChange`, standing record with a target other than the record's `B`: refused in `resume`. Correct.
- `BeginAnchorChange`, no-op: lock released through `Close`, no record written. Correct.
- `BeginAnchorChange`, `writeRecord` failure on a fresh record: `Close` runs with `freshRecord` still false, so a partial record file could stay. Not a correctness break, because the file lives under the lock directory and a re-run rewrites it.
- `start()`, `A == B` with an upstream carrying another anchor: kind integrate, no `.lyx` move. Correct.
- `start()`, no-op on a dirty weft: refused before the no-op is detected. Consistent with D7, where refusals precede everything.
- `resume` and `changeLanded`: a move lands when the marker at `HEAD` is `B`; an integration also needs `U` as an ancestor; a record with no `U` has not landed. Matches D7.
- `recoverUnlanded`, both `.lyx` directories present: refused before anything is touched. Correct.
- `recoverUnlanded`, only `B/.lyx` present: reset to `HEAD`, then the directory is renamed back. Correct.
- `Apply`, `landed` false: `rollback` runs restore, `.lyx` back, rewire at `A`, remove the record, and stops on the first failing step with the record kept. Correct.
- `Apply`, failure after landing: returns `true` and the re-run text, with no restart. Correct.
- `moveWeft`, `moveWeftContent` returning an empty sha: rolled back. Correct, except for the case in finding 1.
- `moveWeftContent`, no tracked names and no `.lyx`: commits the marker alone. Correct.
- `moveWeftContent`, a name's destination already present: refused before any move. Correct.
- `moveWeftContent`, a failed `git mv` after an earlier one succeeded: returns empty and `rollback` restores all pathspecs. Correct.
- `integrate`: the record is rewritten with `U` before the branch moves, and a rollback removes it. Correct.
- `integrateWeft`, a local commit conflicting with the upstream: the build is in a temporary index, so the weft is untouched; the error names the tag and reset steps. Correct.
- `integrateBuild.replay`, a change already in the build: skipped. Correct.
- `replay`, a commit that touched the marker: skipped whole, as D10 says.
- `replay`, a mode-only change: see finding 4.
- `reroot`, a destination path already tracked: refused. Correct.
- `reroot`, `from == "."`: `anchorJoin` and `ls-files` with literal pathspecs give the right paths. Correct.
- `weftAnchorStop`, `reset == false`:
  - Head ahead of the upstream: no stop, so an unpushed re-anchor passes. Correct.
  - Head behind with a different marker: stop, wanting the upstream's anchor. Correct.
  - Diverged with the local marker moved off the merge base: stop, wanting the local anchor. Correct.
- `weftAnchorStop`, `reset == true`: any difference stops, wanting the upstream's anchor. Correct.
- `refuseDropLocalAnchorChange`: an empty marker is refused first, then the comparison. Correct.
- `Pull`: an `ErrAnchorChange` weft stop returns before the warp arm, so warp is untouched. Correct.
- `retryWeftPush`: the empty-marker stop comes first; the move-record stop applies only when the upstream has commits the local tip lacks; then the marker comparison. Matches D7 and D10.
- `classifyFailure` and `record`: an anchor stop is recorded as the `anchor_change` class with its way forward. Correct.
- `Finish`: removes the record, then spawns the push at the pair's marker read through `NamedWorktree`. Correct.
- `Close`: removes a fresh record only if `Apply` never ran; safe to call twice. Correct.

**Findings**

1. **LOW, design** — `moveWeftContent` rolls back a change whose commit already landed.
   - Where: `internal/fabricengine/anchormove.go` (the `CurrentSHA` error return right after the commit) and `internal/fabricengine/anchorchange.go` (`moveWeft`, `if sha == ""`).
   - Case: the commit succeeds, then `CurrentSHA()` fails.
   - What happens: the function returns `"", err`, and `moveWeft` treats that as "not committed". `rollback` then moves `.lyx` back to `A`, rewires `A` and removes the record, while the weft's `HEAD` already holds the marker `B` and the content at `B`.
   - Result: the worktree is wired at `A` against content at `B`, with no record to recover from.
   - Likelihood: very low, since this is a git read failing right after a commit.

2. **LOW, design** — a held move lock always reports the record's pid as the holder.
   - Where: `internal/fabricengine/moverecord.go`, `MoveInProgress`, the `!acquired` branch.
   - Case: a new run holds the lock and has not yet rewritten the record, so the record still carries a killed run's stale pid or a zero pid.
   - What happens: the stops built on this function (weft pull, replaying push, origin reconcile, dobby gate) print "in process N" with a wrong or zero N.
   - Contrast: D7 says to name the run as starting when the record's pid and start time are not the live holder's. `heldMoveRefusal` in `anchorchange.go` does this; `MoveInProgress` does not.

3. **LOW, design** — an own marker that exists but cannot be read silently resolves to the Bolt marker or `"."`.
   - Where: `internal/lyxcwd/anchor.go`, `readRecordedAnchor`, `if data, readErr := os.ReadFile(ownMarker); readErr == nil`.
   - Case: the marker is a directory, or has no read permission.
   - What happens: the read error is swallowed. This is the rewire-at-repo-root outcome D2 says a truncated marker must never produce. D2 and the acceptance criteria only name an invalid or empty marker, so this is a gap rather than a contradiction.

4. **LOW, design** — the integration drops a mode-only local commit.
   - Where: `internal/fabricengine/integrateweft.go`, `replay`, `case current == moved.newBlob`.
   - Case: a local commit changes only a file's mode (the blob is the same).
   - What happens: the comparison sees the blob already present, the commit is treated as already held, and it is dropped into `Skipped` without carrying the mode change.

I did not read `treeAnchor`, `committedMarker`, `refuseUnsettledMoveSource`, `resetWeftToIntegrated` or `restoreMovePathsToHead`, so the cases that depend on them are unchecked.

**Tool calls (all six used)**
1. Read `internal/lyxcwd/anchor.go` lines 140–215: the tail of `readRecordedAnchor` was outside my context.
2. Read the `internal/hubgeom/dobbygeom.go` diff: the card 16 geometry code was not in my context.
3. Read `internal/fabricengine/moverecord.go`: the move record and `MoveInProgress` were not in my context.
4. Read `internal/fabricengine/anchorchange.go` lines 1–330: `BeginAnchorChange` and recovery were not in my context.
5. Read `internal/fabricengine/anchorchange.go` lines 330–597: `Apply`, rollback, `Finish` and `Close` were not in my context.
6. Read `internal/fabricengine/integrateweft.go`, `anchormove.go`, `anchorstop.go` and the `pull.go`, `originreconcile.go` and `pushclassified.go` diffs: the integration, move and stop code was not in my context (the last two tool-call lines cover calls issued in parallel).
