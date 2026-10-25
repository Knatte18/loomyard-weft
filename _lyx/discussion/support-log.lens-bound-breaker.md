```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked every bound, guard, ordering promise and claim about existing code in the decision record. Eight claims broke (two BLOCKING), six held.

**Findings**

1. BLOCKING — D5 and D8: the marker written for a branch with no marker can name a directory where none of its content is.
   - Counterexample: before migration, beam P (anchor ".") has a task child T with no marker of its own, so T's content sits at `_lyx/`. P's anchor is then changed to `x` with `lyx fabric anchor` (D7 allows children and leaves them alone). Reconcile, or an adopt of T, writes "its parent pair's anchor" first, so `x`. Nothing is moved, so T's junctions get wired at `x/_lyx` and point at an empty weft directory, while T's records stay at `_lyx/`.
   - The value whose content is actually there is the one the reader resolved for T before: the Bolt legacy value, or wherever the wired names sit in T's weft tree. The parent's anchor is only right when the parent was never re-anchored.

2. BLOCKING — D10 with D2: a pull that brings an empty marker rewires the pair to the repo root.
   - D2 makes an empty own marker count as absent, so the reader falls back to Bolt, and after the Bolt marker is removed it gets ".".
   - D10 says a pull that "changes the checked-out branch's marker rewires the pair". With a truncated marker on a subpath pair, that unwires the real anchor and wires at ".". This is exactly the second-junction-set damage `refuseEmptyAnchorMarker` exists to prevent, and D8 makes that guard refuse only inside reconcile.
   - Checkout's comparison in D10 has the same hole, and so does Add forking from a pair whose marker is empty. Every path that acts on a marker change or comparison needs the empty-marker refusal, not reconcile alone.

3. MEDIUM — D7 step order: the quiet checks run before the dobby is ended, so there is a race window.
   - Between the refusal checks and step 1, the target's dobby can start a pass: `spawnPass` runs `batten run` with the prime's anchor as cwd and writes batten records under the prime's anchor.
   - For the prime, that pass's records and cwd move out from under it in step 3. Nothing is held across check and move. A `lyx orch start` or `reed up` in the gap opens a session the same way.
   - Fix: end the dobby first and then check, or hold a lock that `reed up` and pass starts respect.

4. MEDIUM — D7 recovery claim: "before the step-3 commit the worktree is at A".
   - D1 reads the marker from the working tree. A kill -9 after the marker file is written (or after `git mv` and the `.lyx` rename) but before the commit leaves the working tree reading B while HEAD says A.
   - Every reader then resolves B, reconcile wires at B over a staged, uncommitted move, and later anchor-scoped commits at B do not stage the deletions at A.
   - Either write the marker as the last act before the commit, or have the re-run detect an uncommitted marker change and finish or roll it back.

5. MEDIUM — D7 refusals don't stop B from lying inside a junction or link.
   - `_lyx/sub` or `.lyx/scratch` "exists as a directory in the worktree's warp checkout" because `_lyx`, `.lyx` and `.scratch` are links into the weft. `ValidateAnchorRel` accepts them, and the warp-pristine check only looks for a real wired name at B.
   - Moving the weft `_lyx` into `_lyx/sub/_lyx` nests the records inside themselves.
   - B must be refused when any segment of it is a junction, the scratch link, or under a hub-reserved or fabric-managed name.

6. MEDIUM — D7 step 3 on Windows: the verb renames a directory it holds open itself.
   - The verb's own logger sink is opened under `<A>/.lyx/logs` (`logger/sink.go` opens the file with `os.OpenFile`).
   - On Windows, renaming `A/.lyx` with that handle open fails every time. On Linux, a later `MkdirAll(<anchor>/.lyx/logs)` recreates a real warp-side `.lyx` after step 2 has removed the junction (the mechanism is in `fabricengine/doc.go` around line 169).
   - The open-risk line on Windows understates this: it is a certain failure of the step as designed, not an untested risk. The sink has to be closed or redirected before the rename.

7. MEDIUM — D3: the scan doesn't catch the pattern it is meant to keep out.
   - Most of the sites D3 lists copy the anchor as `filepath.Join(WorktreePath(l, slug), l.AnchorRel, …)`: `WarpJunctions`, `LauncherDir`, `launcherSpawnRel`, `taskAnchor`, `ideengine` at `spawn.go:70`, `configcli`, and `fabriccli` at `fabric.go:730`. None of them is a `Location` composite literal.
   - A struct copy plus a `WorktreeName` reassignment, or a literal that leaves `AnchorRel` out (zero value, which resolves to "."), also slips past.
   - So the rationale "a constructor plus a scan keeps it from coming back" doesn't hold. The scan must also cover `AnchorRel` reached through any Location other than the one a function was handed for that same worktree, or the claim should be narrowed.
   - `testkit/locationkit` is a non-test file setting `AnchorRel`, so the scan needs an exemption the record doesn't name.

8. MEDIUM — the PATTERN change and D1: "Resolving… a worktree's anchor: only `internal/lyxcwd`" contradicts the record's own decisions.
   - D4 (clone reads `main-weft`), D7 (reads the previous marker value from history), D8 (origin census through `ls-tree`) and D10 (checkout reads the target branch's marker) all read markers from git objects inside `fabricengine`. Clone, reconcile and codebinding already read the marker outside `lyxcwd` today.
   - Without a carve-out for "fabricengine writes markers and reads them from git objects", reviewers will flag D4, D7, D8 and D10 as PATTERN violations.

9. MEDIUM — D8: removing the Bolt marker ignores machines still on the old binary.
   - The census proves every branch is marked, not that every machine runs a binary that reads the per-weft marker. Production reaches each machine only when the operator runs `update-plugins` there.
   - An old binary on the second machine (OSL-1046) reads an absent Bolt marker as "." on a subpath hub, and its `lyx fabric reconcile` wires a second junction set at the root.
   - Open risks should name this, and the removal should be bounded by it.

10. MEDIUM — D8: the census may never complete, so the goal "no marker stays on Bolt" may never be met.
    - Orphan `-weft` branches on origin, whose pair is gone or which no machine checks out (left behind by a remove on another machine, for example), are never migrated, because migration only covers local weft worktrees.
    - The Bolt marker then stays forever. The record should either count only branches some pair holds, or let reconcile commit a marker onto origin-only branches.

11. LOW — D7 re-run: "reading A from the marker's previous committed value".
    - If the worktree had no marker before the change (it fell back to Bolt), the marker commit has no previous value, so the re-run can't find A and the junctions and launchers at A are left behind.
    - D7 should refuse a worktree with no marker (migrate first), or take the fallback value as A.

12. LOW — D7 prime refusals miss other processes running with the prime's anchor as cwd.
    - `hibernatecli` spawns its daemon there (`deps.go:68`) and `adoptcli` starts its gate suite there (`deps.go:116`). Both outlive the verb's checks and lose their cwd anchor.
    - `creelcli` loads reed state from `<anchor>/.lyx` (`delivery.go:272`), which the move relocates while other worktrees' sessions and the creel daemon are live.

13. LOW — D3 and teardown: "needs neither the warp nor the weft directory" plus the D2 fallback gives the wrong anchor once the weft is gone.
    - If a remove that was interrupted after the weft went is resumed, `taskAnchor` and the driver-lock path resolve to Bolt or "." instead of the beam's `X`. The probe then reads the lock as free and the quiet check passes falsely.

14. LOW — D7 step 7: "start the dobbies step 1 ended through the start path `lyx beam start` uses".
    - That path starts the prime's dobby and every beam dobby this machine owns, including ones that weren't running before. And if the verb fails between step 1 and step 7, the dobbies stay down, which the record doesn't say.

15. LOW — D10 checkout: if the target branch has no marker, which value it is compared against isn't defined.
    - After the Bolt marker is removed, the fallback is ".", and a legitimate switch on a subpath hub gets refused, or a wrong one allowed.

**Claims that held**

- Constraint "the weft is never a merge participant": landing doesn't touch the parent's weft.
- D9: the census keyed by anchored pattern, run inside the exclude lock.
- D10 promote: a parent with the same anchor gives an identical marker.
- `revert.go` only resolves for diff (no weft revert verb exists), so it is no extra marker-changing path.
- `taskbranchdiscard` only archives and deletes the branch.
- `parentlock` lives under `BoardDir`, not under any anchor.

**Tool calls**

- Read the headers of `revert.go` and `taskbranchdiscard.go`, to check whether either is another path that changes a weft marker; my context didn't have their contents.
- Read `fabricengine/doc.go` lines 160–180 and the logger sink's `OpenFile`/`MkdirAll` lines, to check whether the verb's own log handle and log-dir recreation sit under `A/.lyx`; my context didn't have the sink's open behaviour.
- Grepped for consumers of the prime's anchor, to find processes the prime re-anchor's refusals miss; my context only covered dobby and teardown.
- Grepped `hibernatecli`, `adoptcli`, `battencli` beam land, creel and reed for `AnchorPath`, to confirm which of those processes run with the prime's anchor as cwd or keep state there; my context didn't have those call sites.
