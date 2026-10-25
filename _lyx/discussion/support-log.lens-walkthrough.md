```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. I ran each decision's mechanism step by step on one concrete case and checked the end state against the purpose the decision states.

**Findings**

1. **BLOCKING — D3 re-entry against D5's ways forward.** "Capture again" via `lyx loom goto Drift-Baseline` does not capture again in two of the halt cases D5 sends there.
   - Case "tree moved before first await": the capture record is `passed`. D3's re-entry rule is "a running or passed capture is left alone", so no new capture starts. Drift-Await is entered again, finds `passed`, a tree that differs and no `awaited_at`, and is Stuck again. The run loops between the same halt and the same way forward.
   - Case "process gone while the record says `running`": D3 re-entry sees `running`, does not check whether the process is alive, and leaves it alone. Drift-Await halts on the dead process again.
   - Evidence: D3's last paragraph and D5's bullets. The fix is to make D3 restart a capture whose process is dead or whose passed tree differs from HEAD's (or to name a different way forward), and to say which of those cases a re-capture can still serve.

2. **MEDIUM — D6 ordering of compare and test gate.** If the `drift-verify` gate fails after a passing compare, the session is re-prompted and the agent changes code again. Nothing re-runs the compare after that gate-driven fix, so the row can end Done on a tree that was never compared.
   - Evidence: D6 says "When the agent ends, the row runs the compare mechanically again… A passing compare is followed by the `verify` gate… re-prompting the session up to three times; then Done."
   - The record should either re-run the compare after each gate pass that followed a code change, or put the gate before the final compare.

3. **MEDIUM — D4/D5 timeout bound against the slot wait.** It is unclear what time Drift-Await's bound ("the capture's own timeout plus one minute") and `capture.timeout_min` are measured from.
   - The detached capture can spend unbounded time waiting for a gate slot, since other worktrees' tier runs hold them.
   - If either clock starts at process start, a busy hub makes Drift-Await give up, or the capture time out, while the command never ran.
   - D5 also never says what Drift-Await does when the bound is passed: Stuck, and with what way forward.
   - Evidence: D4 "holds one slot… for the command's whole run", and D5's `running` bullet.

4. **MEDIUM — D3/D4/D6: nothing commits `_lyx/drift/`.** `_lyx` in the task worktree is a symlink into the `-weft` records worktree. The per-transition status commit (`internal/statuscommit`) commits only the run's status file.
   - The record names no commit path for `scope.json`, `baseline.md`, `verify-report.md` or `_lyx/drift/logs/`.
   - The capture log is written by a detached process after Drift-Baseline has already returned, so no row's seam is positioned to commit it.
   - D4's purpose that the logs "survive the worktree's teardown with the run's records" is unmet unless some row or seam commits them. Drift-Verify, or Finalize at the latest, would have to commit the logs together with the reports.
   - Evidence: `ls -la _lyx` shows the symlink to `regression-baseline-verify-weft/_lyx`, and `statuscommit.go` says it "commits a run's status file".

5. **LOW — D4 dirty check and the baseline directory.** D4 refuses a worktree whose tracked files are dirty, which holds here because `_lyx` points outside the tree. But `{baseline_dir}` sits under the worktree's `.lyx/drift/`.
   - LoomYard's `.gitignore` ignores `.lyx/`. The record does not say that a C# hub like Models must ignore it too.
   - If Models does not, the baseline files show as untracked dirt to the hub's own git status and to Models' `_source.txt` dirty flag. The flag is harmless by #609's trust rule, but verifytree's dirty-tree refusal may count untracked files.
   - Evidence: D4 and the `PATTERN-gate-slot` rule "No site verifies a dirty tree". The record should state the transient exclude, which per `PATTERN-gitrepo-client-boundary` is `.git/info/exclude`.

**Walkthrough list**

- **D1** (Models hub, capture `dotnet test … -e NORCE_BASELINE_DIR={baseline_dir}` with `dir: src/csharp/NORCE.Models`): purpose met. Skip-file support depends on Models, which the record already lists in Open risks.
- **D2** (loom, flag off): Drift-Baseline is Skipped; Drift-Await gets no scope record and returns Done; Drift-Verify is Skipped. Purpose met. With the flag on, the edges Discussion-Bouncer→Drift-Baseline→Plan-Write, Plan-Bouncer→Drift-Await→Webster and Code-Bouncer→Drift-Verify→Describe chain correctly, the plan-perch loop included. Purpose met.
- **D3** (loom, scope `partial` skip A, then a crash and resume): the first pass is met. Re-entry fails the halt cases, as in finding 1.
- **D4** (capture passes, exit 0 on an all-Inconclusive run): met, apart from finding 3 on the timing and finding 4 on the log's survival.
- **D5** (passed capture, tree unchanged; then a Webster reroute after code commits): met through `awaited_at`. The tree-moved and dead-process halt cases are not met, as in finding 1.
- **D6** (compare fails, agent accepts B, fixes C and commits, compare passes, gate fails, agent fixes again): the final tree goes uncompared (finding 2).
- **D7** (accept B during an agent session, then at a Drift-Verify halt, then resume): the current row stays Drift-Verify, so accept is allowed and the next compare skips B. Purpose met.

**Tool calls**

- `ls -la _lyx` plus `.gitignore`: to learn whether `_lyx/drift/` lies in the code tree and how `.lyx` is ignored. My context had neither.
- `internal/statuscommit/statuscommit.go` header: to learn what the per-transition commit covers, which my context did not show.
