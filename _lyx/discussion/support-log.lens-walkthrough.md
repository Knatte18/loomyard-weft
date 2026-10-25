```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens on the umbrella-anchor decision record: I ran each decision on one concrete case, end to end.

**Findings**

1. **MEDIUM — D5 (and D6): the new pair's anchor is read from disk before the weft exists.**
   - Summary: Add can build the new pair's paths before the fork has written its marker to disk.
   - Case: `lyx fabric add T --parent B`, where beam B is at `X` and the prime at `.`.
   - Evidence: D3's constructor reads `<hub>/T-weft/.lyx-anchor` from the working tree. The fork is `worktree add --no-checkout`, and the files arrive only at the later `checkout-index`. Any Add step that builds paths for T before that (the warp worktree, launchers, the run-records root) gets D2's fallback (Bolt or `.`), not `X`. With `--anchor` the value on disk is also wrong until the move.
   - Fix: D5/D6 should say the new pair's anchor comes from the fork point (the parent's Location, or `opts.Anchor`), not from a fresh read.

2. **MEDIUM — D8 and D5's adopt rule: a markerless branch gets the wrong value once its parent has been re-anchored.**
   - Summary: the value order (parent's anchor, then the prime's, then Bolt legacy) can name a different anchor from where the branch's content sits.
   - Case: the hub anchor is `.`. Beam B is migrated, then moved to `backend` with D7. Task T was forked before migration, is still markerless, and sits on origin from another machine. Adopting or migrating T writes `backend`.
   - Evidence: T's content is at `_lyx`, the legacy hub anchor. Junctions at `backend/_lyx` would point at an empty directory.
   - Fix: a markerless branch predates per-worktree anchors, so its content is at the legacy hub anchor. Take the Bolt legacy value first, or find the anchor directory that holds the wired names in the branch's tree. Use the parent's anchor only when neither answers.

3. **MEDIUM — D7 step 1: ending a dobby also ends the runs it is driving.**
   - Summary: ending a beam's dobby, or every dobby when the prime moves, kills the batten passes those dobbies are running for child tasks, and does nothing on Windows.
   - Evidence: `runproc.End` sends SIGTERM to every process registered under the owner "and the processes below them". A dobby's in-flight pass is a child `lyx batten run <child>`. The refusals only check batten runs of the target worktree, or of the hub when the target is the prime. On Windows, `End` signals nothing, so the dobby keeps running while its `.lyx/beam/` directory is renamed under it.
   - Fix: refuse while a dobby pass is in flight, and either stop only the dobby process or refuse a live dobby with a way forward. State the Windows behaviour.

4. **MEDIUM — D7 validation: `B` can resolve through a junction.**
   - Summary: "a directory in the worktree's warp checkout" lets `B` sit under `_lyx`, `.lyx` or `.scratch`.
   - Case: `--set _lyx/config`.
   - Evidence: `os.Stat` follows the junction, so the path counts as a directory and passes `ValidateAnchorRel`. The pristine check looks for a real `_lyx` at `B`, finds none, and passes. Step 3 then moves `_lyx` into its own subtree.
   - Fix: `B` must be a directory git tracks in the warp, with no segment that is a junction, `.lyx` or the scratch link. The same applies to D6's `--anchor`.

5. **MEDIUM — D8: the Bolt marker may never be removed.**
   - Summary: the census condition ("every weft branch on origin carries a marker") can stay false indefinitely.
   - Evidence: an origin weft branch that no live machine has checked out (an abandoned task, a retired machine's pair) is never migrated, because D8 migrates only local weft worktrees. Separately, a pass's own marker commits are not on origin until pushed, so removal can only happen on a later reconcile. The record states neither.
   - Fix: state that removal happens on a later pass once the pushes land, and say what migrates or ignores branches with no worktree anywhere. Otherwise "no marker stays on Bolt" is not met.

6. **LOW — D7 failure path: the ended dobby is left down.**
   - Evidence: step 7 restarts it, but nothing in the failure paths does. The refusal or way forward should name `lyx beam start`.

7. **LOW — D7 re-run rule: a `--set` equal to an inherited anchor looks like a resume.**
   - Case: a task forked from a beam at `backend`, then `--set backend`. The marker already reads `B`, so the verb takes the resume path. It reads `A` from the marker's previous committed value, which is the beam's pre-create `.`, and runs the cleanup steps against that.
   - Fix: a settled worktree (junctions at `B`, none at `A`) should be a no-op.

8. **LOW — D10's pull rewire.**
   - Summary: the pull rewire runs with none of D7's checks for a live session, run or dobby.
   - Evidence: agents in that worktree lose their anchor cwd mid-turn, and the dobby's `.lyx/beam/` directory moves under it. The record should state this bound or accept it explicitly.

9. **LOW — D3's scan: `internal/testkit/locationkit/locationkit.go` sets `AnchorRel` in a literal.**
   - Evidence: it is a non-`_test.go` file, so "production code outside lyxcwd" would flag it unless testkit is exempted.

10. **LOW — D3: a gone pair's anchor.**
    - Evidence: with the weft directory gone, the constructor returns the fallback. After the Bolt marker is removed that is `.`, which can be wrong for teardown paths of a subpath pair. This is not among the open risks.

**Per-decision walkthrough**

| Decision | Case | Purpose met |
|---|---|---|
| D1 | Resolve at `<hub>/B` with `B-weft/.lyx-anchor` = `backend`; the gate needs cwd `<hub>/B/backend` | Yes |
| D2 | Markerless pair with Bolt `backend` gives `backend`; with neither, `.` | Yes |
| D3 | Teardown and dobby geometry for a beam at `backend` from a prime at `.` | Yes for live pairs; not for a gone pair (finding 10) |
| D4 | Fresh `clone --subpath backend`, then a re-clone adopting it | Yes |
| D5 | `add --parent` to a beam at `X` | Not met (findings 1, 2) |
| D6 | `beam create --anchor Y`, plus a re-run with the pair present | Met for the re-run; the fresh create hits finding 1, and validation has finding 4's gap |
| D7 | Beam `.` to `backend` while its dobby drives a child pass | Not met (findings 3, 4, 6, 7) |
| D8 | Bolt `backend` hub with pairs on two machines | Migration met; Bolt removal not met (findings 2, 5) |
| D9 | Unwiring B at `backend` while P is at `.` | Yes |
| D10 | Checkout to a branch with a different marker refuses; a pull from A to B rewires; promote refuses a mismatch | Yes, with finding 8 |

**Tool calls**

- Read `internal/runproc` End's doc and signature, to check whether ending a dobby also ends the processes below it; my context did not hold the runproc semantics.
