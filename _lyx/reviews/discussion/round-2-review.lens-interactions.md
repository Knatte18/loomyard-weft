```yaml
lens: interactions
outcome: reported
```

**Pairs checked**

- D2 empty-marker refusal × D8 per-pair reconcile refusal — held.
- D5 (`Add` takes anchor from the parent's Location) × D7 (marker written last, then committed) — **broken** (F2).
- D6 × D7 (D6 reuses D7's move lock, record and re-run recovery) — held.
- D7 step 1 (record holds the pre-integration tip, then a long wait) × D7 recovery ("an integration landed when `HEAD` differs from the tip the record holds") — **broken** (F3).
- D7 step 1 wait × D7 steps 2–4 × acceptance "waits … for a held parent lock" — **broken** (F1).
- D7 prime steps 1–2 (wait covers runs parented to the prime; step 2 ends every dobby of this machine) × acceptance "waits, with no pass started meanwhile" — **broken** (F4).
- D7 × D5 markerless value (D7's `A` for a markerless worktree) — **broken**, LOW (F6).
- D7 step 3 junction removal × D9 exclude census — held.
- D7 × D3 constructor reading the working-tree marker during a move — held for the verb's own steps; the `Add` case is F2.
- D7 prime change × D4 clone adopting from `main-weft` — held.
- D7 prime change × D5 inheritance for later forks and the D3 portal sweep — held (stated).
- D7 child-keeps-anchor × D10 promote refusal — held (children forked before the change are refused on promote, way forward named).
- D7 move record × D8 reconcile refusal and `restoreWeftLockDir` — held.
- D7 step 6 launchers × reconcile `restoreLaunchers` — held while the record stands.
- D7 step 2 spared pushes × D10 push stop and the recovery tests — held.
- D10 pull/push stop condition × D7's integrate condition and way forward — held (both ways forward reach D7's integrate branch).
- D8 migration commit × D10 pull/push stop against a markerless upstream — **broken** (F5).
- D8 Bolt removal × D2 fallback on other machines — held (Open risks).
- D7 step 2 `EndSparing` × Constraint "dobbies end only through `runproc.End` or `EndSparing`" — held.
- D7 step 2's "one a dobby spawned itself is ended with it" × acceptance "sparing the pushes and board syncs" — **broken**, LOW (F7).
- D7 step 8 (start only a beam this machine owns) × D7's owner refusal — held.
- D7 re-run with another `--set` refused × D10 stop naming a later anchor — held.

**Findings**

F1 — BLOCKING — design — D7 steps 1–4 and acceptance "waits … for … a held parent lock"
Summary: D7 reads the parent lock once and never holds it, so a child's landing can commit into the weft while it moves.
Evidence: D7 step 1 waits until the parent lock of the worktree's branch "is free", then proceeds; nothing in steps 2–8 takes or holds it. A child loom run is not a batten pass — it runs on its own driver between dobby passes (`internal/dobbyengine/doc.go:36`), so "starts no pass" does not stop it. Its Finalize takes the parent lock (`internal/loomcli/landingdeps.go:375-382`) and, before its parent-side merge, commits the parent pair's run records through `parentRecordsCommitter` (`internal/loomcli/landingdeps.go:117`, `:465-471`), resolving the parent at call time — reading `A` or `B` from the half-moved working-tree marker. D7's own one-weft-commit move (step 4) and its recovery tests assume no other commit on the weft while the record stands.
Fix: hold the parent lock of the worktree's branch from step 1 to step 8 (or refuse while a child loom run is live).

F2 — MEDIUM — design — D5 × D7 (and D6 from a moving parent)
Summary: nothing refuses an `Add` from a worktree whose move record stands, so a child can get anchor `B` while its content sits at `A`.
Evidence: D7 step 4 writes the marker `B` last, then commits; a kill between that write and the commit leaves the working-tree marker `B` while the branch `HEAD` carries `A`. D5 has `Add` take the new pair's anchor from "the parent's Location", which by D3/D2 reads the working-tree marker (`B`), while the fork itself is cut from the branch (marker `A`, content at `A`). D7 gates only `EnsureLive`, the dobby loop and reconcile on the record; `lyx fabric add`, `lyx beam create` and adopt's step-2 `Topology.Add` are not gated.
Fix: refuse `Add` from a worktree whose move record stands, naming the re-run.

F3 — MEDIUM — design — D7 step 1 record × D7 "landed" test for an integration
Summary: the recorded tip goes stale during step 1's wait, so a re-run after a kill can read an integration that never ran as landed.
Evidence: step 1 writes "the weft's tip before it" into the record, then waits, possibly long, on child passes and the parent lock; weft commits can still land meanwhile (the Finalize records commit of F1, spared pushes that replay). `HEAD` then differs from the recorded tip before step 4 ever starts, so a kill before step 4 reads as landed and the re-run renames `A/.lyx` to `B/.lyx` and wires at `B` with content at `A`.
Fix: record the tip at the start of step 4, after the wait, or test "landed" as `HEAD` descending from `U` and carrying marker `B`.

F4 — MEDIUM — design — D7 prime steps 1–2 against the refusal "any live batten run"
Summary: for the prime, step 2 can end a beam dobby's pass mid-flight that neither the refusal nor the wait saw.
Evidence: the prime refusals run before step 1, while beam dobbies still start passes; step 1's wait covers only runs whose parent worktree is the prime; step 2 ends every dobby of this machine with its descendants. A pass started after the refusal check for a beam-parented run is a ppid descendant of that beam's dobby (`spawnPass`, `internal/beamcli/dobbydeps.go:464-481`) and `planRecords` walks every descendant (`internal/runproc/end.go:316-343`), so step 2 ends it. The same window exists on a beam between the dobby's record check and the pass taking its driver lock (`internal/battencli/drive.go:63`), since the wait reads that lock. This breaks the acceptance "waits, with no pass started meanwhile, for a child run's pass in flight".
Fix: for the prime, wait on the passes of every run of every dobby step 2 ends, and treat a pass the dobby spawned but whose lock is not yet taken as in flight.

F5 — MEDIUM — design — D8 migration commit × D10 pull/push stop
Summary: D8's own migration commit stops every pull and push of a pair whose upstream gained commits from a not-yet-migrated machine, and the integration the stop names is undefined for that case.
Evidence: D10 stops a pull or replaying push when "the upstream has commits the local tip lacks while the two tips carry different markers"; only D10's checkout clause reads a markerless tip at D5's value, the pull/push clause does not. After D8 commits marker `X` locally, an upstream that gained markerless records commits (old binary, or a machine whose reconcile has not run) reads as a different marker, so D8's spawned detached push stops, and every later pull and push of that pair, `main-weft` included. D10's integration then applies "D7's move from `U`'s marker", but `U` has none.
Fix: read a markerless tip at D5's value in D10's pull/push comparison and in the integration's source anchor.

F6 — LOW — consistency — D7 `A` × D5 markerless value
Summary: D7 takes `A` for a markerless worktree from `lyxcwd`'s fallback, not from D5's value.
Evidence: D7 moves from "its checked-out anchor `A`", which for a markerless weft is `lyxcwd`'s D2 fallback (Bolt, else `"."`), while D5 says the anchor of a markerless branch "wherever one must be written or compared" is the shallowest directory holding a wired name, and D7 is not among D5's listed uses. On a census-missed pair once the Bolt marker is gone, D7 moves from `"."` while content sits elsewhere.
Fix: name D7 among D5's uses, or refuse a markerless worktree naming `lyx fabric reconcile` first.

F7 — LOW — consistency — acceptance "sparing the pushes and board syncs" × D7 step 2
Summary: the acceptance criterion overclaims what D7 step 2 spares.
Evidence: D7 step 2 concedes that a push a dobby spawned itself "is ended with it", and `planRecords`' descendant walk applies no spare check to children (`internal/runproc/end.go:326-343`), so a push or sync descending from an ended dobby is ended even with a spared role.
Fix: narrow the acceptance to pushes and syncs outside the ended dobbies' process trees.

No tool calls made.
