```yaml
lens: walkthrough
outcome: reported
```

**Walkthrough list**

- D1 — worktree `x` resolves its anchor from `<hub>/x-weft/.lyx-anchor` through `weftname.SiblingPath` — purpose met.
- D2 — own marker absent / empty / invalid, then Bolt fallback, then `"."` — purpose met.
- D3 — the prime building pair `p`'s paths after `p` was re-anchored — purpose met.
- D4 — fresh `fabric clone --subpath backend`, then a second clone adopting `backend` from `main-weft` — purpose met.
- D5 — beam `P` killed in D7 step 4 after writing marker `B` but before the commit, then `lyx fabric add --parent P` — purpose **not** met (F1).
- D5 markerless — a markerless branch whose content sits at `backend/_lyx`: the shallowest-directory rule yields `backend` — purpose met.
- D6 — `beam create --anchor Y` from the prime at `.`, killed before the move commit, re-run — purpose met.
- D7 — beam `X` from `A` to `B` with a child loom run live and its batten at Run-Shed between continue passes — purpose **not** met (F2).
- D7 prime — prime `A`→`B` while beam `Q`'s dobby starts a pass for a `Q` child between the refusal check and the step-1 record — purpose **not** met (F3).
- D7 recovery — kill after step 4 renamed `A/.lyx`→`B/.lyx` but before the commit, re-run — purpose **not** met (F4).
- D7 — first run with an uncommitted file under `A/_lyx` in the weft — purpose **not** met (F5).
- D7/D10 integration — machine 2's prime with local records commits at `A`, upstream marker `B`: replay, re-root, branch move and `.lyx` rename chain cleanly — purpose met.
- D7/D10 integration kill before/after the branch moved: the landed test on `HEAD` vs recorded tip holds — purpose met.
- D8 — migration of a markerless prime then pairs, then the Bolt census with an unpushed marker: removal deferred — purpose met.
- D9 — two worktrees at the same anchor, one unwired: line kept — purpose met.
- D10 checkout — pair at `A` checking out a target weft branch at `B`, following the way forward — purpose **not** met (F6).
- D10 integration conflict — conflict on local commit `c2` of `c1..c3`, following the way forward — purpose **not** met (F7).
- D10 promote — task at `Y`, parent at `X`: refused with `--set X` — purpose met.

**Findings**

- **F1 — MEDIUM — design — D5 / D7 (Add from a worktree with a standing move record).** D5 takes a new pair's anchor from the parent's Location, which `lyxcwd` reads from the working-tree marker (D1). D7 step 4 writes the marker `B` last, then commits; a kill between the write and the commit leaves the working-tree marker `B` and `HEAD` at `A`. Until the re-run, `fabric add`/`beam create`/adopt's `Add` from that parent forks the weft from `HEAD` (content and marker at `A`) but wires the pair and builds its origin record at `B`. D7 gates only `EnsureLive`, the dobby loop and reconcile on the move record; `Add` from the moving worktree is never refused. Evidence: decision-record.md:78 (D5), :133 (step 4 order), :124–126 (what the record gates).
- **F2 — BLOCKING — design — D7 step 1 (parent-lock wait is a single read; nothing holds the lock).** The entry requires "No run may be in flight in the worktree while it moves." A child loom run lives in its own pair, independent of batten passes; between continue passes no batten driver lock is held (dobbyengine/doc.go:36), so step 1's wait sees no pass in flight and a free parent lock and proceeds. The child's Finalize can then take the parent lock and commit the parent's run records into the moving weft through `CommitParentRecords` (loomcli/landingdeps.go:117, :465–471) — the parent's anchored pathspec at `A`, during or after steps 3–5 — and merge into the parent. The verb holds neither `internal/parentlock`'s lock nor the weft write lock across steps 2–8. Evidence: decision-record.md:127, parentlock/doc.go (Acquire; valid while owner lives).
- **F3 — MEDIUM — design — D7 steps 1–2 for the prime (wait covers fewer runs than step 2 ends).** For the prime, step 1 waits only for runs whose parent worktree is the prime, but step 2 ends every dobby of the machine and their descendants. Beam dobbies spawn their passes as children (beamcli/dobbydeps.go:464–481), and `planRecords` walks ppid descendants (runproc/end.go:316–343). The "any live batten run" refusal runs before step 1 writes the record, so a pass beam `Q`'s dobby starts in that window — or one spawned before the dobby saw the record but not yet holding its driver lock (battencli/drive.go:63) — is killed mid-pass, possibly mid-Land into `Q`. Evidence: decision-record.md:110, :118, :124–129.
- **F4 — BLOCKING — design — D7 "Not landed" recovery order destroys `.lyx`.** The wired name set includes `.lyx` (fabricengine/junctionnames.go:303, `structuralNeverCommittedDirs`), so `B/<names>` covers `B/.lyx`. The Not-landed branch first "deletes what `HEAD` does not track under `B/<names>`" — `B/.lyx` is never tracked, so it deletes the renamed machine-local state (logs, locks, strand state) — and only then "renames `B/.lyx` back to `A/.lyx` when only `B/.lyx` exists". A kill after step 4's rename and before its commit thus loses the worktree's `.lyx` on re-run. Evidence: decision-record.md:133, :149–150.
- **F5 — MEDIUM — design — D7 steps (first run has no dirtiness check).** No refusal or step on a first run checks the weft for uncommitted or untracked files under `A/<names>`; only the recovery path "runs the dirtiness check on the rest of the weft". Step 4 moves only tracked content and commits "all of it": an uncommitted modification is swept into the marker commit, and an untracked records file is left at `A`, outside every junction after step 5. "It skips no dirtiness check on the weft" names no check that runs. Evidence: decision-record.md:110–118, :133, :151, :158.
- **F6 — MEDIUM — design — D10 checkout way forward re-anchors the branch being left.** Checkout from a branch at `A` to a target at `B` names `lyx fabric anchor --set B` first. That verb commits `B` and moves the content on the current weft branch — the one being left — which is shared with every other machine checking it out, so their pulls/pushes stop until each runs an integrating `--set` (D7 last paragraph, D10). The way forward permanently rewrites an unrelated branch to make a branch switch possible. Evidence: decision-record.md:161–162, :190.
- **F7 — MEDIUM — design — D10 integration-conflict way forward drops later local commits.** On a conflict at local commit `c2` of `c1..c3`, the way forward saves `c2`'s changes and `git reset --keep c2^`, which also drops `c3` from the branch; `c3`'s changes are neither saved nor written back, and the re-run integrates only `c1`. `c3`'s records are lost from the branch (only the reflog keeps them). Evidence: decision-record.md:200–201.

**Tool calls**

1. `junctionnames.go` `RepoWiredNames`/`structuralNeverCommittedDirs` grep — to learn whether `.lyx` is in `<names>` for D7's recovery order (not in the manifest).
2. `junctionnames.go` lines 25–60 and `WiredNames` — same purpose, the name-set composition.
3. `junctionnames.go` `junctionNames` — to confirm the union includes `.lyx`.
