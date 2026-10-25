```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: I traced the code that already runs against each decision's new state and checked whether the result survives it.

**Findings**

1. BLOCKING. D3, D4, D6 and Constraints: nothing commits `_lyx/drift/`.
   - `_lyx` is the junction into the records repo (fabricengine doc). Agents never commit records, and only the loop owner does, through the commit, discussion, plan and status seams.
   - The record names no seam for `scope.json`, `baseline.md`, `verify-report.md` or `logs/`.
   - So "survives the worktree's teardown with the run's records" (D4) does not hold. Teardown's `logArchive` copies only the trace and verify sources, and the uncommitted files go with the worktree.
   - D5 also assumes the scope record outlives an adopt; uncommitted, it does not.
   - The capture writes its log from a detached process while other rows' seams commit records. The record must say which row commits these files and when; for example `Drift-Await` commits the capture log once the capture has ended, never mid-write.

2. BLOCKING. D5 and D3: a re-capture after code has changed.
   - `awaited_at` lives in `.lyx/drift/capture.json`, which is ephemeral and machine-local.
   - Take an adopt on another machine, or a lost `.lyx`, after Webster began. Webster's reroute through the plan perch re-enters `Drift-Await`, which finds no capture record and halts with "`lyx loom goto Drift-Baseline` to capture again before any code changes".
   - That goto re-enters `Drift-Baseline`, which starts a capture whenever the record is absent or failed (D3). D4's only guard is a dirty check on uncommitted files, so a capture runs on Webster's committed code and blesses its drift.
   - The guard should be HEAD equal to the task's branch point (its merge base with the parent), not only a clean tree. `awaited_at`, or a "code began" fact, belongs in a durable record.

3. MEDIUM. D4 and D6: a hibernate or pause during a regression run.
   - `verifytree.Verify` honours a covering pause: it writes a marker, SIGTERMs at the grace end and reruns after resume.
   - The drift capture and compare are not `Verify` calls, so this machinery does not cover them. A wall-clock `timeout_min` straddling a machine sleep fires on wake and records `failed`, and `Drift-Await` halts on it.
   - The record is silent on pause and hibernate for the detached capture.

4. MEDIUM. D3: amendments after Discussion never reach the scope.
   - `lyx loom decision add` amendments route to `amendment_target: Plan-Burler-Review`, past `Drift-Baseline`.
   - A scope decided on the pre-amendment record is never revisited, and an amendment that makes a refactoring change the model leaves scope `full`, so `Drift-Verify` fights intended drift.
   - The record should say the `Drift-Verify` agent judges against the amended record, or that an amendment does not re-scope.

5. LOW. D7: the halted run's current row.
   - `lyx drift accept` refuses unless the run's current row is `Drift-Verify`.
   - After a Stuck halt the status names the halted row, so this probably holds. But the operator's way forward "accept and resume" depends on it, and the record should state it.

6. LOW. D2: `entry_gate: drift.enabled` on a hub-wide module.
   - Every precedent reads `budget.<Segment>` or a per-worktree module.
   - Whether loom's and darn's `Env.Setting` resolves a `HubWide` module was not verified.

7. LOW. D2, darn: darn's Code perch changes code after `Drift-Verify`.
   - Darn's `Code-Burler-Fix` commits after `Drift-Verify`, so its fixes are never compared. This is the entry's order, but the record states the loom-side consequence and not the darn-side one.

**Paths traced**

- D2, `Shed.enter`'s skip routing for the two gated rows: survives.
- D2, a run profile setting `code_max_bounces` to 0 still routes `Code-Bouncer`'s Done to `Drift-Verify`: survives.
- D2, row-name status resume for in-flight runs: survives.
- D3 and D4, the loop-owner commit seams: break (finding 1).
- D4, teardown's `logArchive` sources: break (finding 1).
- D4, `pairteardown`'s `EndSparing` killing the capture: survives, since `Drift-Await` halts on the dead record.
- D4, gate-slot holder records and `Holders` readers: survive.
- D4, the verifytree pause machinery: breaks (finding 3).
- D5, adopt or lost `.lyx`, then Webster's reroute to `Drift-Await`: breaks (finding 2).
- D5, `lyx loom goto Drift-Baseline` after Webster: breaks (finding 2).
- D6, `verifytree.DirtyPaths` excluding ignored `.lyx` and the `_lyx` junction: survives.
- D6, Finalize's reroute to `Parent-Align`, the Code perch and `Drift-Verify` with a fresh budget: survives.
- D3, the amendment route past `Drift-Baseline`: breaks (finding 4).
- D7, the halted row as the current row: probably survives (finding 5).

**Tool calls**

1. `lyxdirs` doc and the cleanup `RemoveAll` sites: what removes `.lyx` and `_lyx` state.
2. `pairteardown/archive.go` and the fabricengine junction doc: what survives teardown, and whether `_lyx` is the records junction.
3. verifytree's dirty and pause handling, and the archive's source list: dirty-tree reads and which logs are archived.
4. burlerengine's overlay doc and a commit-seam grep: who commits records.
5. Commit-seam locations: where the seams live.
6. `loomcli/wiring.go` seams: what the status and discussion seams commit.
