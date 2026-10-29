```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens on the `pr-review-row` decision record. I checked each pair of decisions that act on the same state, and checked the acceptance criteria and constraints against the decisions they touch.

**Pairs checked**
- D1×D2, how the mode resolves from the parent's `beam.yaml` plus the board field: held.
- D2×D4, a field `on` with no GitHub origin: the halt's way forward is cancel and rerun, which is consistent with the field being frozen by the lock: held.
- D2×D3, the Warn for `off` under `required`, with Finalize calling the same resolver: held.
- D3×D6, a no closes the record's open pull request and removes the record: held.
- D3×D11, the two handlings of a no while a record stands: broken (F4).
- D3×D12, a no closes the pull request and removes the record, and the teardown then finds none: held.
- D3×D13, the pawl row and the beam reading its parent's anchor: held.
- D4×D11, Finalize re-evaluates the origin facts through the resolver: held.
- D6×D8, the first push recorded as the gated head, then the gate moves it: held.
- D6×D9, a recorded pull request that GitHub reports closed: held.
- D6×D11, a halt with no record leads to `goto PR-Review`, and a closed record also halts: held.
- D7×D9 pause, the session is stopped and the call ends transient, then the next call resumes the session by its id: held.
- D7×D9 approval branch, whether a session the operator closed gets resumed: broken (F5).
- D7×D9 merged branch, stopping the session before the final read: broken (F2).
- D7×D12, runproc and the teardown: held.
- D8×D9, Done only on the gated head: broken on the merged branch (F1).
- D8×D10, the gate's verify mark, with the `PR #&lt;n&gt;` mark set again after it: held.
- D8×D11, Finalize's push through the D6 push and the detached-push bound: held.
- D9 enumerated states × D8 dirty tree: broken (F3).
- D9×D10, the wait mark cleared on every way out, pause and cancel included: held.
- D9×D11, approval followed by Finalize's push and a stale-approval halt: held, since Open risks names it.
- D9 merged × D11 already-landed path: held for a clean task head, broken under F2.
- D11×D12, a no landing locally with a record left standing: broken (F4).
- AC D8 and D9 against the D8 bound: no criterion covers an ungated head merged on GitHub (folded into F1).
- The constraint that GitHub refuses an author's approval, against D5 and D9: held, since Open risks names it.

**Findings**

- **F1 — MEDIUM — consistency — D8 Bound and D9's first merged bullet.**
  D8's bound says D9's Done reads only the record's gated head.
  D9's merged branch compares the merged head with the task head instead.
  Counterexample: the agent commits, and the detached records push carries that ungated head to the pull request.
  Someone merges it on GitHub, the merged head equals the task head, and the run is Done.
  Finalize then takes the already-landed path.
  So the bound's "lands nothing until the row has gated it" is false for this case, and no acceptance criterion covers it.
  Evidence: `_lym/discussion/decision-record.md` D8, the Bound line ("D9's Done and D11 read only the record's gated head"), and D9, the first bullet ("merged, with the merged head equal to the task head").

- **F2 — MEDIUM — consistency — D9 merged branch against D9 approval branch and D7.**
  The approval branch stops the session before its last head and tree read, so that a commit made after the approval stays out of the landing.
  The merged branch reads head and tree with the session still live and stops it only at Done, under D7.
  An agent commit made between that read and the stop leaves a task commit the merge lacks.
  Finalize's parent-side merge then lands it locally, on top of the GitHub merge, without it ever being on the pull request.
  Evidence: `_lym/discussion/decision-record.md` D9, the rationale's last line and the merged bullet; D7 ("stops it ... at Done").

- **F3 — LOW — consistency — D9 state list.**
  "Merged, with the merged head equal to the task head" but a dirty tree matches no bullet.
  The Done bullet needs a clean tree, and the lacks-commits bullet needs extra commits.
  So the run's behaviour after a merge on GitHub with uncommitted agent edits is undefined.
  Evidence: `_lym/discussion/decision-record.md` D9 bullets.

- **F4 — LOW — consistency — D3, D11 and D12.**
  The row's no closes the record's open pull request at once and removes the record.
  Finalize's no lands locally and leaves the open pull request to the teardown.
  D12 then says "a landed run's pull request is merged and left as it is", which is false for a run that landed locally.
  The teardown closes the pull request with a comment naming the teardown's cause rather than the local landing.
  Two mechanisms act on the same record with different semantics.
  Evidence: `_lym/discussion/decision-record.md` D3 ("A no ... closes it ... and removes the record"), D11's first bullet and D12's second sentence.

- **F5 — LOW — consistency — D7 and the D9 approval branch.**
  D7 says the wait loop starts no session, so an operator who closes the window ends the agent until the next entry.
  The D9 approval branch's "anything else resumes the session (D7)" runs inside that wait loop, and resumes a session the operator closed whenever an approved head moved or the tree is dirty.
  Evidence: `_lym/discussion/decision-record.md` D7 ("The wait loop of D9 starts no session") and D9's approval bullet.

Tool calls: none; the inherited context held every file this lens needed.
