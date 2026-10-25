```yaml
lens: way-forward
outcome: reported
```

**Way-forward lens: each refusal's next step, and whether it progresses**

- Card 1, `LoadConfig` key refusals (`{baseline_dir}` missing, `timeout_min`, `verify_max_runs`, model): next step is `lyx config drift`. Progresses: configcli writes a hub-wide module's file at the board dir from any worktree (`internal/configcli/doc.go:3,12`).
- Card 1, `Config.Step` on an undeclared step: next step is `lyx config drift --set '&lt;name&gt;={...}'`. Progresses, for the same reason.
- Card 5, `RunCapture`/`StartCapture` when the lock is held: wait (`lyx drift status`), then `lyx loom resume`. Progresses. At a `Drift-Await` halt the resumed row waits on the lock and then reads the record. At `Drift-Baseline` the row waits (darn) or leaves the capture running (loom).
- Card 5, `CheckMainCapture` wrong row: it names where the capture runs and `lyx loom status`. Progresses once the run reaches that halt.
- Card 5, `CheckMainCapture` moved HEAD: `lyx drift defer`, then `lyx loom resume`. Progresses. `Defer` admits a blocked `Drift-Await` and a `Drift-Baseline` halt opening with `CaptureHaltPrefix`, and the card 9 Stuck carries that prefix.
- Card 5, `CheckMainCapture` dirty tree: clean the paths, then `lyx loom resume`, "which captures again", or defer and resume.
  - Progresses at a `Drift-Baseline` halt.
  - Does **not** progress when the refusal comes from `lyx drift capture` at a `Drift-Await` halt (F1).
  - "Removing" a tracked path does not clean it (F2).
- Card 5, `StartCapture` when the child exits before its record, or the record wait runs out: an error naming the child's log. Inside the row (card 9) the resumed entry captures again, so it progresses. On the `lyx drift capture` CLI the error names no next step (F3).
- Card 6, `RunCompare` when the marker is held: wait, then run it again. Progresses.
- Card 7, `RecordScope` once a scope record exists: write `baseline.md` alone and end the turn. Progresses: the re-entered row skips the agent once both files exist.
- Card 7, `RecordScope` wrong row or invalid combination: names the row and `lyx loom status`, or the correct invocation. Progresses.
- Card 7, `Accept` in a state other than running or blocked: `lyx loom resume`. Progresses.
- Card 7, the agent session's `Accept` of a withdrawn test: fix the drift, or report it in `verify-report.md`. Progresses.
- Card 7, a caller's `Accept` outside a decision halt (withdrawn test, or scope `full`): the parent's ruling at a decision halt. Progresses. Under `full` every blocked Drift-Verify halt is a decision halt. Under `partial`, resuming to a failing final compare with a non-empty withdrawn list yields one.
- Card 7, `Withdraw` of a test not on `accepted`: `lyx drift status`. Progresses.
- Card 7, `Defer` at the wrong halt: names that halt's own ways forward. Progresses.
- Card 7, `Defer` while the lock is held: `lyx loom resume`. Progresses. `Drift-Await` waits on the capture and then passes or halts again, and `Defer` is then admitted. At the `Drift-Baseline` capture halt the loom row leaves the capture running and returns Done.
- Card 7, `Waive` at a halt that has a counted capture: `lyx loom resume` plus that halt's own ways. Progresses.
- Card 7, `CheckVerifyRow`: names `Drift-Verify`. Progresses.
- Card 7, `ErrRunsSpent` for the agent session: write the report and end the turn. Progresses: the turn end runs the compare gate entry, and a failing compare is a terminal Stuck that names accept or fix.
- Card 7, `ErrRunsSpent` for a caller at a halt: `lyx loom resume` gives a fresh agent and a fresh budget. Progresses. The "fresh agent" wording is inaccurate when the resumed compare passes, but the run still moves on.
- Card 9, agent ended with no scope record: `lyx loom resume`, which spawns the agent again. Progresses: no scope record means the agent is spawned.
- Card 9, undeclared capture: `lyx config drift`, then resume. Progresses.
- Card 9, refused capture (`CheckMainCapture` with the prefix): defer and resume, or clean and resume. Progresses at `Drift-Baseline`, apart from F2.
- Card 9, failed foreground capture (darn, and the loom Windows fallback): `lyx drift capture` then resume, or defer then resume. Progresses. `lyx drift capture` is admitted at the `Drift-Baseline` row. On resume the darn row waits on the lock, and the loom row leaves the capture running, or leaves a passed record alone.
- Card 9, any other `StartCapture` error: returned as an error, and the resumed entry captures again. Progresses.
- Card 10, Stuck with HEAD on the branch point (missing, failed or dead capture): `lyx drift capture` then resume, or defer then resume. Progresses: the resume waits on the new capture and passes on a passed record at HEAD. A dirty tree blocks the capture step (F1).
- Card 10, Stuck after HEAD moved: defer, then resume. Progresses: `awaited_at` set means Done.
- Card 11, no counted capture: `lyx drift waive --reason` then resume. Progresses: `Waive` is admitted, and the next entry returns Done at step 1.
- Card 11, undeclared compare: `lyx config drift`, then resume. Progresses.
- Card 11, step 6 dirty tree: commit or clean, then resume. Progresses.
- Card 11, step 6 failing test gate: fix the tests, commit, then resume. Progresses, but only by the caller's own fix, since step 6 runs before any agent spawn.
- Card 11, failing final compare: `lyx drift accept` then resume, or fix then resume. Progresses. An accept skips the test in the next compare. A fix re-runs step 6 and then the compare.
- Card 11, decision halt with `unruled` acceptances: resume to stand by them, or `lyx drift withdraw` then resume. Progresses. On the resumed entry step 2 rules the survivors before the compare.
- Card 11, decision halt for an acceptance left by an error stop: same ways forward. Progresses.
- Card 13, capture-job kill error: stops the teardown, with no next step named (F4).
- Card 14, entry build refusals: each names the config fix. Progresses.

**Findings**

**F1. MEDIUM, consistency.** Cards 5 (`CheckMainCapture`), 7 (`lyx drift capture`) and 10.
- The dirty-tree refusal tells the caller to clean the paths and run `lyx loom resume`, "which captures again".
- `CheckMainCapture` is also the precondition of `lyx drift capture` at a `Drift-Await` halt. There, `lyx loom resume` re-enters `Drift-Await`, which only reads records. Card 10 says it waits on a held lock, or is Stuck on a missing, failed or dead record. It never captures.
- So the named step halts again on the same missing or failed capture. The way forward that works there is to clean the tree and run `lyx drift capture` again, then resume.
- Evidence: `_lyx/plan/05-capture-run.md:61-66` (one refusal text, shared by the row and the verb); `_lyx/plan/07-drift-verbs-cli.md:65` (`capture` without `--side` calls `CheckMainCapture` at a `Drift-Await` halt); `_lyx/plan/10-drift-await-producer.md:24-30` (`Drift-Await` never captures).

**F2. LOW, design.** Cards 5 and 9, and the Shared Decision "a dirty capture halt names cleaning".
- The dirty-tree way forward offers "removing" the listed paths.
- The listed paths are tracked dirty paths (D4: "a worktree whose tracked files are dirty"). Deleting a tracked file leaves a tracked deletion, which is still dirty, so the resumed capture is refused again.
- Only stashing, or restoring the file to HEAD, clears it.
- Evidence: `_lyx/plan/00-overview.md:64` ("cleaning the listed paths (removing or stashing them)"); `_lyx/plan/05-capture-run.md:65`; `internal/fabricengine/diff.go` `CodeDirtyPaths` reads tracked uncommitted changes, deletions included.

**F3. LOW, consistency.** Card 5 `StartCapture` and card 7 `lyx drift capture`.
- A child that exits before its `running` record, or a record wait that runs out, returns "an error naming the child's log" with no named next step.
- The decision record's Constraints require every refusal and halt to name its way forward. From the operator's CLI nothing says what to do next, for example re-running `lyx drift capture` after acting on the log's refusal, or `lyx drift defer`.
- Inside the row (card 9) the resumable error does progress.
- Evidence: `_lyx/plan/05-capture-run.md:70-71`; `_lyx/discussion/decision-record.md:245`.

**F4. LOW, design.** Card 13 `EndSession`.
- A `killCapture` error "stops the sequence with no seize and no session end", and the card prescribes no way forward.
- `TerminateNamedJob` fails on Windows on a persistent cause, such as an access-denied open. Every retry of `lyx fabric remove`, batten's teardown or adopt then fails at the same step, leaving the pair impossible to tear down, and the error names no step that gets past it.
- Evidence: `_lyx/plan/13-capture-teardown.md:24-26,30`; `internal/proc/namedjob_windows.go:66-84` (an open or terminate failure returns an error); `internal/pairteardown/teardown.go:298` (the owned-end error precedent is wrapped with no way forward).

Tool calls:
- `grep internal/configcli/*.go` for hub-wide write behavior. My context did not show whether `lyx config drift --set`, run from a task pair, writes the hub file. Card 1's and card 9's way forward depend on it.
