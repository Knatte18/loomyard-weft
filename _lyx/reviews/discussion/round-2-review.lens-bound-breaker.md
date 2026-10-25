```yaml
lens: bound-breaker
outcome: reported
```

**Claims attacked**

1. D7 step 1: the wait "no pass in flight, read as its driver lock held" plus the dobby starting no pass once the record stands keeps passes from being killed — **broken** (F1).
2. D7 prime: the step-1 wait covers every pass step 2's ending of every dobby on the machine reaches — **broken** (F2).
3. D7 step 1: the parent lock read free once means a child's landing into the worktree has finished and none starts during the move — **broken** (F3).
4. D7: the move record keeps `lyx orch start`, `lyx adopt` and `lyx beam start` from starting work — **broken** (F4).
5. D7 prime refusals: a live creel daemon is refused, and every refusal names its way forward — **broken** (F5).
6. D7 step 2: `EndSparing` spares pushes and syncs outside the dobby's tree and ends the ones it spawned — held (the descendant walk in `planRecords` ignores `skip`, matching the record's concession).
7. D7 Windows: a live dobby is refused since `End` signals nothing — held.
8. D7 recovery: an integration landed when `HEAD` differs from the record's tip — weak but held: while the record stands the only other weft writers are spared pushes, which D10 stops for an integration (both tips' markers differ); a child Finalize's parent-records commit could also move `HEAD` (covered under F3).
9. D10: a pull only ahead is not stopped; a diverged pull compares both tips — held.
10. D8 census: an unfetched, lagging, empty-marked or unpushed ref errs toward keeping the Bolt marker — held; the never-fetched case is a stated open risk.
11. D7: only the prime's board owner can start a prime change, via the upstream-marker refusal — held (`PrimeOwner` on the board, `internal/boardengine/board.go:304`).

**Findings**

- **F1 — BLOCKING — design — D7 steps 1–2: a pass started in the window between a dobby's last record check and its driver-lock acquisition is killed.**
  Counterexample: dobby reads "no record", spawns `lyx batten run` (`internal/beamcli/dobbydeps.go:464-485`); the verb writes the record and samples `DriverLock(prime, slug)` — free, because the pass is still in `battenPreRun` and takes the lock later (`internal/battencli/drive.go:60-80`); step 2 then ends the dobby and the pass with it, since `planRecords` walks every ppid descendant (`internal/runproc/end.go:316-345`). Nothing in the record makes the dobby's record check and spawn atomic against the verb's write-and-sample.

- **F2 — BLOCKING — design — D7 prime: in-flight passes of beam-parented runs are killed.**
  Step 1 waits only on runs "whose parent worktree is this one" (the prime), but step 2 ends "every dobby this machine runs", whose passes are their descendants. The prime refusal "any live batten run" runs before step 1, so a beam dobby starting a pass between that refusal and the record write is neither refused nor waited on, and is killed — possibly inside Land/Finalize. Evidence: `internal/beamcli/dobbydeps.go:464-478` (passes are children of the dobby), `internal/runproc/end.go:316-345`.

- **F3 — BLOCKING — design — D7 step 1: the parent-lock check is a single sample; the lock is never held.**
  A child's loom run is a separate process, not inside a batten pass (between continue passes no batten driver lock is held, `internal/dobbyengine/doc.go:36`). Its Finalize takes the parent lock and commits the parent pair's run records before merging into the parent (`internal/loomcli/landingdeps.go:117`, `:375-382`, `:465-471`). Counterexample: verb sees the parent lock free, proceeds to step 4; the child's loom Finalize then acquires the lock and runs `CommitParentRecords` on the moving weft (anchor-scoped pathspecs) while content is between `A` and `B` and junctions are removed, and may land into the parent's branch. For a beam no refusal catches this: the reed refusal covers only the worktree's own strands, and the child's driver strand is in the child's pair. The move takes neither the parent lock nor the weft write lock for its duration.

- **F4 — MEDIUM — design — D7: `lyx adopt` is bounded only on dobby starts; its other effects run during a move unbounded and unstated (attack-surface rule).**
  While the record stands, adopt adds pairs through `Topology.Add` from the prime, resumes runs as detached first passes through `battencli.StartFirstPass` (in-process, not through dobby, `internal/battencli/cli.go:580`), spawns `lyx loom resume` in child pairs, and starts `lyx gate test --gate finalize` in the prime (`internal/adoptcli/doc.go` steps 2, 5, 6). During a prime move these write batten records and gate state under `A` or take a gate slot mid-move. D7's list of unguarded starters names a session, a hand pass and a hand-started `lyx beam dobby`, not adopt's resumes, suite or `Add`.

- **F5 — MEDIUM — design — D7 prime refusals: the creel-daemon refusal rests on a false premise and has no way forward.**
  Creel's state is `<hub>/_board/.lyx/creel`, not under the prime's anchor (`internal/hubgeom/creelgeom.go:24`, `internal/fabricengine/junctionnames.go:167`); the daemon "serves the hub ... until it is stopped or its binary is replaced" (`internal/creelcli/cli.go:769`), and no `lyx creel` stop verb exists (verb list in `internal/creelcli/cli.go`). Any post starts it, so a prime change is refused indefinitely with no lyx way forward (`PATTERN-refusal-way-forward`).

- **F6 — MEDIUM — design — D5/D7: `Add` from a worktree with a standing move record is not refused.**
  D7 writes the marker last before commit; a kill between that write and the commit leaves the working-tree marker `B` while `HEAD`'s marker is `A`. A later `lyx fabric add`/`lyx beam create` from that parent (or adopt's `Add`, F4) takes the new pair's anchor from "the parent's Location", which `lyxcwd` reads from the working tree — `B` — while it forks from the branch commit, whose marker and content are at `A`. The record gates `EnsureLive`, the dobby loop and reconcile on the record, not `Add` (decision-record D5 line 78, D7 lines 124-126, 133).

**Tool calls**

1. `internal/battencli/drive.go:50-80` and `StartFirstPass` in `internal/battencli/cli.go` — to see when a pass takes the batten driver lock and whether adopt's resume goes through dobby; the manifest had only the lock's readers, not its holder's timing or adopt's resume path.
