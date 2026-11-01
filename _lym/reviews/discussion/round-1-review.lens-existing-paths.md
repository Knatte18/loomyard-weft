```yaml
lens: existing-paths
outcome: reported
```

Existing-paths lens: trace each state the decision record introduces or changes through the code that already reads, writes or ends it.

**Paths traced, per decision**
- D1, the per-key run directory under `gateslot.WaitDir`:
  - `gateslot.ReadWaits` reads only the `wait-*.yaml` glob (`internal/gateslot/wait.go:62-90`), so new subdirectories are ignored. Its readers `loomcli/reviewstatus.go:179`, `hubgeom/webstergeom.go:42`, `driftcli` and `cardverify.go:100` all survive.
  - The green record (`gatecli/run.go:71`) shares the directory. The runner runs in-process with `--wait`, so the record's lock and the clean-tree rule are unchanged. Survives.
- D1, the runner's own wait record: `acquireSlot` writes a wait record under the runner's PID (`run.go:541`). The status and review-status readers see a live waiter. Survives.
- D1 and D3, a runner ended by an existing path before it writes its exit file. **Breaks** (finding 1):
  - `runproc.End`/`EndSparing` sends SIGTERM, then SIGKILL after the grace (`runproc/end.go:19,53`).
  - It is called on the slug by the tender verb (`tendercli/deps.go:321`), by tender's owner end (`deps.go:113`) and by pair teardown (`pairteardown/teardown.go:603`).
  - A crash, an OOM kill or a reboot ends the runner the same way.
- D2 and the Constraint "Go gate calls carry `--wait`":
  - `verifytree.Verify` strips the strand and runs `--wait` calls (`verify.go:294,382`).
  - Plan context refuses a card Verify line without `--wait` (`loomshed/plancontext.go:143`).
  - `cardverify.IsGateCall` requires `--wait`, and any other line runs inside a held slot (`cardverify/runner.go:101-127`).
  - The loom verify gate renders `--wait` calls (`loomshed/verifygate.go:35`).
  - All survive.
- D3, registration under the slug: pair teardown spares only named roles, so the new gate role is ended unless it is added to a spare list. Survives as the record intends. `lym tender` lists records from `runproc.Owners` (`tendercli/deps.go:105-118`), so a registered runner is listed. Survives.
- D4, `lym shuttle await` config read: today the verb is skipped by `PersistentPreRunE` resolution (`shuttlecli/await.go:18`). D4 states the new worktree resolution and the fallback outside a worktree. Survives.
- D5, reading run records:
  - `ReadRunRecords` skips undecodable files and treats a missing directory as empty (`runlog.go:128-154`).
  - `pruneRunLog` drops records older than 14 days, so "latest five" means within 14 days.
  - The median's exclusion of an abandoned call reads a property no record field carries. **Breaks** (finding 2).
- D5, the new call id against `lym gate report`: `report.go:90` groups by key and tier list, and an optional field decodes tolerantly. Survives.

**Findings**

1. BLOCKING, design. Location: D1 join rule and D3 lease.
   - Summary: a front joins whenever the lease holds, the result is undelivered and the tree matches. Nothing checks that the runner process is alive, and D3 does not say that a runner ended by a terminating signal writes its exit file.
   - Counterexample: `runproc.End` escalates to SIGKILL (`internal/runproc/end.go:19,53`). It runs on the slug from `internal/tendercli/deps.go:321`, `deps.go:113` and `internal/pairteardown/teardown.go:603`; a crash, OOM kill or reboot has the same effect.
   - Result: the runner leaves no exit file. Each re-run renews the lease (D3), so the agent gets exit 75 `done: false` indefinitely and no fresh run starts.
   - The record states no bound for a dead runner. The fix is to record the runner's PID and start time and check its liveness on join, and to state the runner's exit-file write on SIGTERM.
2. MEDIUM, design (deferred fact). Location: D5 median filter.
   - Summary: the median leaves out "an abandoned call", but `gateslot.RunRecord` has no field that marks abandonment or a refusal (`internal/gateslot/runlog.go:38-73`), and D5 adds only a call id.
   - Gap: how a tier killed by the lease's cancel is recorded, and so how the reader tells an abandoned call from a full one that failed, is left for the implementer to find out.
   - Risk: a call abandoned inside its last listed tier, with a short wall time, would count as full and lower the median. The record should name the field or rule that identifies an abandoned or refused call.

Tool calls:
1. Grep for readers and sweepers of the gate wait directory: needed every reader of D1's new directory tree.
2. `pairteardown` end and spare code, and `tendercli` End callers: needed the paths that end the slug's processes.
3. `gateslot/wait.go` `ReadWaits`, and the tender verb context: needed the glob that decides whether subdirectories break readers.
4. `runproc/end.go` SIGTERM/SIGKILL sequence, and the dobby env: needed to know whether the end path can kill the runner before it writes its exit file.
5. Grep for Go sites that spawn `lym gate test`: needed to check the `--wait` Constraint against the existing callers.
