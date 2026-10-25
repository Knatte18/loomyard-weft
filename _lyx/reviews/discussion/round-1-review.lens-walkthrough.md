```yaml
lens: walkthrough
outcome: reported
```

Task: walkthrough-lens review of the umbrella-anchor discussion pair. Each decision is run end to end on one concrete case.

## Walkthrough list

- D1 — worktree `foo`, marker read at `<hub>/foo-weft/.lyx-anchor` through `weftname.SiblingPath`: purpose met.
- D2 — weft with no marker on a hub whose Bolt says `backend`, then an empty own marker: purpose met as written.
- D3 — pair torn down after its weft is gone: purpose met. `Remove` cleans launchers before the git removal (`remove.go:141`), so no anchored path is needed after the weft is gone.
- D4 — fresh `clone --subpath backend`, then a re-clone: purpose met on the write side. The probe does not support the fresh-bind wording (finding 10).
- D5 — child forked with `--parent` from a beam at `X`, then the parent re-anchored, then migrated: purpose met. Ties at one depth cannot occur, because a markerless branch has its content at one anchor.
- D6 — `beam create --anchor Y`, killed after the fork and before the move commit, then re-run: purpose partly missed (finding 9).
- D7 — beam `b` from `.` to `backend`: purpose missed. Step 1 ends the passes (finding 1). The recovery cannot tell its states apart from the record (findings 4, 5, 6).
- D7 for the prime: purpose missed (findings 2, 3).
- D8 — two machines on a `backend` hub, one reconciles first: purpose missed for the other machine's prime (finding 7).
- D9 — pairs at `.` and `backend` sharing the exclude file: purpose met.
- D10 checkout — target branch at another anchor: purpose met; the refusal has to come before the warp switch at `checkout.go:83`.
- D10 pull — pair after its own re-anchor, push not landed yet: purpose missed (finding 8). Same pull on a second machine's prime: purpose missed (finding 3).
- D10 promote: purpose met.

## Findings

1. **BLOCKING · design · D7 step 1 (decision-record.md:114, AC :197)**
   `runproc.End` cannot end "the dobby process alone, leaving any pass it spawned to finish".
   `End` signals every process registered under the owner, then walks each record's descendants by ppid and signals them too (`internal/runproc/end.go:316-345`).
   `spawnPass` starts each pass as a direct child of dobby (`internal/beamcli/dobbydeps.go:476-480`; `proc.Detach` only sets a new group), so the pass in flight is killed.
   The acceptance criterion "a pass that dobby had in flight finishes" fails.
   For a beam, owner `b` can also hold that pair's own pushes (`fabricengine/doc.go:756`).

2. **BLOCKING · design · D7 prime refusals vs step 1 (decision-record.md:110, :114-115)**
   For the prime, step 1 ends every dobby this machine runs, which kills every in-flight pass in the hub (finding 1).
   Under `runproc.HubOwner` it also ends the prime's records pushes (`fabricengine/spawn.go:128`, `runproc/record.go:21`).
   Only after that do the refusals run, so "any live batten run" reads a state step 1 has just destroyed.
   The guard kills the runs it exists to protect, and then passes.

3. **BLOCKING · design · D7 "on this machine" with D10's pull refusal (decision-record.md:98, :159)**
   Every machine's prime checks out the same `main-weft`.
   Re-anchoring the prime on machine 1 commits and pushes marker `B` there (step 7).
   On every other machine, the next prime pull brings a different marker and is refused.
   Its way forward, `lyx fabric remove <slug>` then adopt again, is impossible for the prime: `refusePrimeSlug` refuses it (`internal/fabricengine/remove.go:949`).
   The record has no path for the other machines' primes.

4. **MEDIUM · design · D7 recovery trailer rule (decision-record.md:128-129, AC :199)**
   "With no such trailer, as for an anchor inherited through a fork" is false for a child forked after its parent's re-anchor.
   The child's commit that last changed the marker is the parent's trailered commit, so a `--set` equal to the child's inherited anchor is not seen as inherited.
   It cleans the child at the parent's old `A`.
   "A `--set` equal to a settled worktree's anchor is a no-op" is also undefined.
   From the record alone, a run killed after the step-3 commit (marker `B`, trailer `A`) looks the same as a completed change, so a no-op on it leaves no junctions at `B`.

5. **MEDIUM · design · D7 "re-run first undoes an uncommitted move" (decision-record.md:127, :132)**
   Nothing records that a move was in flight, or its `B`, and the re-run may come from another worktree with a different `--set`.
   The never-tracked `A/.lyx` → `B/.lyx` rename leaves no trace in git.
   The killed run leaves the weft dirty, and "it skips no dirtiness check on the weft" would refuse the re-run before the undo.
   Run the other way round, the undo discards uncommitted changes it cannot tell from its own.

6. **MEDIUM · design · D7 recovery claim about reconcile (decision-record.md:126, Notes :216)**
   The claim is false for a pair whose weft is present.
   Such a pair goes to `repairPairWiring` (`reconcile.go:225-229`), which re-wires junctions, the scratch link and launchers at the marker's anchor (`reconcile.go:391-424`).
   `isRawWarpWorktree` runs only on the missing-weft path (`reconcile.go:514`).
   - A reconcile after the step-3 commit wires at `B` and leaves `_launchers/<A>/<slug>`.
   - A reconcile before the commit wires at `A` into content that has already moved.

   The record is silent on reconcile running during an interrupted change, and the plan-writer note points at the wrong function.

7. **MEDIUM · design · D8 census (decision-record.md:140)**
   The census reads origin's remote-tracking ref of `main-weft`, which machine 1's pushed marker satisfies.
   Machine 2's local `main-weft` may not have pulled that commit yet.
   Once machine 2 pulls Bolt with the marker removed, its prime resolves `"."` on a subpath hub.
   The Open risks cover only an old binary (decision-record.md:184-186), not a new binary in this window.

8. **MEDIUM · design · D10 pull refusal (decision-record.md:159)**
   After a pair's own re-anchor, step 7's push is async and may fail.
   Origin's tip still carries `A` against the pair's `B`, so the pull is refused although fast-forward or replay would keep `B`.
   The way forward, remove and adopt from origin, drops the unpushed marker commit.
   The rule should refuse only a pull that would change the checked-out marker.

9. **LOW · consistency · D6 re-run (decision-record.md:90)**
   - Kill after the fork and before the move commit: the re-run refuses and names `lyx fabric anchor <slug> --set <dir>`, but does not say to re-run `beam create` for the create steps still owed.
   - Kill after the move commit and before wiring: "an equal `--anchor` is a skip" does not say whether the wiring at `<dir>` is redone.

10. **LOW · design · D4 fresh-bind test (decision-record.md:69)**
    `probeWeftBinding` does a `--single-branch` clone of HEAD, which is Bolt, and reads markers only there (`recordsprobe.go:73-108`).
    A marker on `main-weft` cannot be counted without a fetch the probe does not do, and the record does not say how it is counted.

Tool calls:
1. `weftname.SiblingPath` and the order of `fabricengine/remove.go`: the manifest did not cover launcher-before-weft removal order or `refusePrimeSlug`.
