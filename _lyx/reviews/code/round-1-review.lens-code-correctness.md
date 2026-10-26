```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens over the fix-plan diff (`planfixer.go`, `coldfixer.go`, `fixplan.go`, `conflictrecord.go`, `runlevel.go`, `strand.go`, `state.go`).

**List: function or branch, case run, result**

- `ReadConflictRecord`, absent file: returns an empty record, so the caller refuses. Correct.
- `ReadConflictRecord`, header `# Plan conflict at begin-batch 02`: `Atoi("02")` gives 2. Correct.
- `ReadConflictRecord`, torn record with a header and no findings: caught by the `len(Findings)==0` refusal. Correct.
- `ReadConflictRecord`, finding text containing a newline: the second line would be dropped or misread. `begin-batch` writes findings verbatim, but they are single-line check strings, so this is not reachable in practice. Correct for the producers that exist.
- `FixPlanSpawnOrAttach`, last record non-terminal: attaches before reading the record. Correct.
- `FixPlanSpawnOrAttach`, last record terminal with the same findings: refused, with a way forward. Correct.
- `FixPlanSpawnOrAttach`, two earlier fixers for the batch with different findings: refused. Correct.
- `FixPlanSpawnOrAttach`, a different batch's earlier fixers: not counted. Correct.
- `FixPlanSpawnOrAttach`, `HeadSHA` or `DirtyPaths` failing: returns a refusal that names the transient way forward, before any spawn. Correct.
- `FixPlanSpawnOrAttach`, the spawn started but `FindRun` fails: the strand is stopped and the stop error joined in. Correct.
- `FixPlanSpawnOrAttach`, the stale-reply `os.Remove`: runs before the spawn, and a missing file is tolerated. Correct.
- `FixPlanAwait`, reply `verdict: fixed`, HEAD unchanged and no new dirty path: `fixed`. Correct.
- `FixPlanAwait`, empty or garbled reply: `plan_writer`. Correct.
- `FixPlanAwait`, `fixed` with HEAD moved or a new dirty path: overruled to `plan_writer` with a warning.
  - Correct: porcelain output includes untracked `??` entries, so a new untracked file is caught.
  - Correct: the plan is reached through a symlink into the weft (`git ls-files` showed `_lyx/plan/...` is beyond a symbolic link), and `rebaseline` fabric-syncs the weft. A legitimate plan edit therefore neither dirties the worktree nor moves its HEAD.
- `FixPlanAwait`, `codeTouchedSince` returning an error after the fixer has ended: returns `(nil, err)`, so the CLI never persists the terminal record. The record stays `Status==""`, and a later `fix-plan` attaches and re-awaits, which is recoverable. Correct, though see F2.
- `awaitColdFixer`, the order of the done, asking, timeout and died checks: identical to the removed `FixVerifyAwait` body. `FixVerifyAwait` still returns `Status`, `DeadReason`, `Reply`, `ElapsedS` and `Warnings`. Refactor correct.
- `reclaimEntryTimeStrands` and `RemoveRecoveryStrands`: a non-terminal plan fixer is stopped, then marked dead/died. Terminal ones are left alone. Correct.
- `fixPlanCmd`, spawn then `SaveState` failing: the strand is stopped and the error joined. Correct.
- `fixPlanCmd`, a `running` result: nothing is persisted and no sync runs. Correct.
- `fixPlanCmd`, a terminal result: `persistFixPlanTerminal` reloads under a fresh lease, so it does not clobber the state `rebaseline` wrote while the lease was released. Correct.
- `fixPlanCmd`, `fabricSync` failing after a terminal result: returns an error although the record is saved terminal. A re-run then refuses on equal findings, which is conservative. Correct.

**Findings**

F1: LOW / design — `internal/websterengine/planfixer.go:174-183` (`codeTouchedSince`).
- A fixer that modifies a source file already uncommitted at the spawn passes the check, because only paths new since the spawn count.
- D7 and the acceptance criteria say "a worktree with uncommitted changes". The code and doc.go narrow that to new paths.
- The narrowing is documented in doc.go and the summary. This is a deliberate weakening, but it is not what D7's text says.
- Evidence: `planfixer.go:174-183` (`slices.Contains(dirtyAtSpawn, path)`).

F2: LOW / design — `planfixer.go:150-156` and `fixplan.go`.
- When `codeTouchedSince` fails on a `git` read after the fixer is done, `FixPlanAwait` returns an error and the CLI exits without stopping the strand or persisting the terminal state.
- The reply file and the record both stay. The next call re-awaits and recovers.
- The error text carries a way forward ("transient, re-run"). This is not a hang.

F3: LOW / design — `planfixer.go` (`FixPlanAwait` and the dead/timeout paths) together with the stencil's restore step.
- If the fixer dies, times out, or is overruled after it has edited cards or after `rebaseline` accepted them, nothing restores the plan. The fixer's own `restore-plan` step is the only restore.
- Master then ends stuck over a plan that holds the fixer's partial or accepted edits.
- On an overrule, those accepted edits stay in the recorded plan, and loom's plan review starts from the edited plan. The D7 text only promises a `plan_writer` verdict, and this is consistent with it.
- Weak, since D3 says restore is the fixer's job; I flag it only as an unaddressed state on failure paths. Not tested.

F4: LOW / design — `planfixer.go:100-110`.
- `st.PlanFixers` persists across run respawn, and so across loom reroutes to plan review.
- After a plan-writer round, the per-batch cap of two and the equal-findings refusal still count fixers from before that round.
- A later, genuinely mechanical conflict at the same batch can then be refused and go straight to stuck.
- D6 says "of this run", which matches the code literally; the effect is conservative.

No BLOCKING or MEDIUM findings from this lens.

**Tool calls**
1. `internal/verifytree/verify.go` (`DirtyPaths`), `git ls-files _lyx`, `git check-ignore`, and a grep of `PlanConflictError`. My context lacked how dirty paths are computed, whether the plan is tracked in the worktree, and what batch number `begin-batch` records.
2. `internal/webstercli/rebaseline.go` and `restoreplan.go`, to confirm `rebaseline` and `restore-plan` use `fabricSync` (the weft) and not the worktree HEAD.
