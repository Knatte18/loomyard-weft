```yaml
lens: existing-paths
outcome: reported
```

Task: existing-paths lens review. I traced the existing code paths each decision's state meets.

## Paths traced, per decision

- D1/D2, the `lyxcwd` marker read, against `preflight`'s stale-marker guidance (`preflight.go:45`) and clone's stale guard (`clone.go:359-369`): **survives**. Both read Bolt only, and D2 keeps `ErrStaleAnchorMarker` on the Bolt read.
- D2/D8, the empty-marker refusal, against `refuseEmptyAnchorMarker` (`reconcile.go:168,282`), which reads Bolt today: **survives** once it is made per pair.
- D4, the fresh-bind test, against `probeWeftBinding` (`recordsprobe.go:75-79,108`), a `--single-branch` shallow clone of HEAD (Bolt): **breaks in part**. The probe cannot see a `main-weft` marker. `freshBind` stays right only because `.lyx-warp` is on Bolt.
- D4, the bootstrap guard (`clone.go:192-200`), which names `.lyx-anchor` as the weft evidence: **misreads** after D4. Its message names a marker Bolt no longer carries.
- D4, the adopt read (`clone.go:372`), which reads the marker from the Bolt worktree on disk: the decision states the change. **Survives** as a requirement.
- D7 step 1, `runproc.End`, walked through `planRecords` (`end.go:296-345`): **breaks**. It signals every ppid descendant of the dobby, so its detached passes die too.
- D7 step 1, `runproc.End(HubOwner)` for the prime, against `pushSpawnCommand` (`spawn.go:128`): **breaks**. It also ends every in-flight records push that has no teardown slug.
- D7 recovery, against `Reconcile` with a present weft (`reconcile.go:217-227`) and `repairPairWiring` (`reconcile.go:391-424`): **breaks the stated premise**. `isRawWarpWorktree` runs only on the missing-weft path (`reconcile.go:514`), so reconcile does act on an interrupted change.
- D7 step 7 and D6, the push of the marker commit, against the push path: **survives**.
- D8, the marker commits, against the census and the push spawn sites (`weft_verbs.go:269`, `pendingflags.go`): **breaks the bound**. Reconcile pushes nothing (`reconcile.go`, no spawn), so an idle pair's marker commit never reaches origin.
- D8, the unwire help text (`fabric.go:420-424`), which already says unwire keeps `.lyx-anchor`: **survives**. The docs constraint covers the wording.
- D9, the census in `namesWiredInSiblingWorktrees` (`junction.go:636-666`): **survives** once each sibling is checked at its own anchor through D3.
- D10, `Checkout` (`checkout.go:84-104`), where the warp switch runs before the weft branch is resolved: **survives** if the refusal runs before line 84. The record says "before switching either side".
- D10, the weft pull on a pair (`pull.go:290-330`, `PullWeft` then the replay in `settleUnmovedWeft`): **survives** for pairs, as a stated requirement.
- D10, the weft pull of the prime on another machine, against `pull.go:249` with D7's prime change: **breaks** (finding 1).
- D10, promote's `replaceRecordsContent` (`destroy.go:2309-2329`): **survives**. `read-tree parentTip` carries the parent's root marker, and the decision refuses that case. `runsRoot` uses the task's `AnchorRel`, which equals the parent's once the refusal has passed.
- D3/D7, the merge geometry in `resolveMergeGeometry` (`mergepaths.go:26-32`): **survives**. The weft never merges, and the warp junctions are untracked.

## Findings

1. **BLOCKING · design · D10 pull, with D7 on the prime.**
   Re-anchoring the prime on its owner machine pushes marker `B` on `main-weft`.
   Every other machine's prime then pulls an incoming marker different from its own and is refused.
   The way forward the refusal names, `lyx fabric remove <slug>` and adopt again, does not exist for the prime.
   The record has no rule for propagating a prime anchor change to the other machines' primes.
   Evidence: `decision-record.md:159`, `decision-record.md:98` ("on this machine"), `decision-record.md:105` (the owner refusal), `pull.go:249-330`.
2. **BLOCKING · design · D7 step 1, and the acceptance criterion at line 197.**
   Step 1 says it ends the dobby process alone through `runproc.End`, leaving its passes to finish.
   `End` walks every ppid descendant of each record and signals it, groups included.
   `spawnPass` starts each pass as a child of the dobby, so step 1 kills every pass in flight.
   The premise about existing code is false, so the "a pass … finishes" criterion cannot hold.
   Evidence: `end.go:316-345`, `dobbydeps.go:476-482`, `decision-record.md:114,197`.
3. **MEDIUM · design · D7 step 1 for the prime.**
   For the prime, `runproc.End` runs under `HubOwner`.
   Every detached records push with no teardown slug is also registered under `HubOwner`.
   So step 1 kills the in-flight pushes of every pair, which the record neither bounds nor refuses.
   Evidence: `spawn.go:128` (`pushSpawnCommand` owner), `dobbydeps.go:64-69`, `decision-record.md:114`.
4. **MEDIUM · design · D7 recovery paragraph.**
   The claim that "reconcile does not repair it, since it treats a pair with no `_lyx` at its anchor as unmanaged" is false for a present weft.
   With a present weft, `Reconcile` runs `repairPairWiring`, which rewires the junctions and the scratch link at the marker's anchor.
   - After a kill past the step 3 commit, reconcile wires `B` and does none of steps 5 to 7.
   - After a kill before the commit, it wires `A` into a weft whose content moved uncommitted to `B`.

   The record is silent on what reconcile does in both windows, and on how the verb's re-run then reads the state reconcile left.
   Evidence: `reconcile.go:217-227,391-424,514,582`, `decision-record.md:126`.
5. **MEDIUM · design · D8 census.**
   Removing the Bolt marker waits for every counted branch's remote-tracking ref to carry a marker.
   Reconcile's marker commits are never pushed by reconcile; weft pushes come only from the detached-push spawn sites.
   An idle pair's marker commit can therefore stay local without end, and the Bolt marker with it.
   That contradicts "no marker stays on Bolt", and the record names no push of the D8 commits.
   Evidence: `reconcile.go` (no push spawn), `weft_verbs.go:269`, `decision-record.md:140-141`.
6. **LOW · consistency · D4 fresh-bind test.**
   D4 says the fresh-bind test counts a marker on `main-weft`.
   The probe is a `--single-branch` shallow clone of HEAD (Bolt) and never sees `main-weft`.
   The bootstrap guard still names `.lyx-anchor` as the weft evidence, so the decision needs the probe to change or the sentence to go.
   Evidence: `recordsprobe.go:75-79,108`, `clone.go:192-200`, `decision-record.md:69`.

Tool calls:
1. `grep` for marker refs in `fabriccli` and others, plus `replaceRecordsContent`: the promote reset and where clone commits the marker were missing.
2. `fabric.go` unwire help and marker grep: the CLI-layer marker text was missing.
3. `resolveMergeGeometry` and raddle writers: the merge path for a child at another anchor was missing.
4. Raddle grep in `merge_verbs.go` and `weftwiring.go`: where raddle is regenerated was missing. Inconclusive.
5. `spawn.go` `pushSpawnCommand`: the push owner that `End(HubOwner)` reaches was missing.
6. `PushSpawn` callers and reconcile push grep: whether anything pushes D8's marker commits was missing.
