```yaml
lens: bound-breaker
outcome: reported
```

Bound-breaker lens: I attacked each bound, guard, guarantee and claim about existing code in the decision record.

**Claims attacked**

1. D8: "Drift-Verify repeats the check as the guarantee". **Broken**, see F1.
2. D7: "never touches the baseline directory". **Broken** in the sense that nothing guards the baseline, see F1.
3. D9: the judge's snapshot of HEADs, tracked tree, untracked paths and `_lym/drift/`. **Broken**, see F2.
4. D9: the judge, a session that died mid-write, and a stale `judgement.yaml`. **Broken**, see F3.
5. D9: what a judge-guard Stuck leaves behind. **Broken**, see F4.
6. D6: the expect re-entry "ended normally, recorded as today's drift producers record (`AgentSessionOpen`)". **Broken**, see F5.
7. D6: "the Warp-SHA shows the expectation existed before any code" and its bound (a hand-made commit "before the writer's first warp commit"). **Broken**, see F6.
8. D6: the expect agent commits on the warp before the seal. **Partly broken**, see F7.
9. D2: under `command` mode, a judge's `does not match` stops the run. **Broken** by a plain resume, see F8.
10. D7: "the lock serialises concurrent checks" together with "inherits a held slot". **Broken** by deadlock, see F9.
11. D1 rationale: a repeated key's shifted occurrence "costs judge work and lets no regression through". **Weakened**, see F10.
12. D4: Done at once at the last passing head with no amendment since. Held: a weft-only edit after a passing check can no longer change containment.
13. D10 and D11: a stored ruling is reused after an amendment or `--discard`. Held, because an amendment drops the rulings of the cells it covers and `--discard` leaves a cell uncovered.
14. D11: the caller check is bounded by the strand-name check and the shell guard. Held.
15. D15: `run_settings` carrying the retired `skip` or `verify_max_runs` seeds, and the load drops the key. Held: `ValidateLocal` admits a retired `CategoryNone` key unchecked (`internal/configengine/layers.go:291-299`), and `refuseRetiredKeys` fires only for keys mapped in `RetiredWayForward` (`internal/configreg/runsettings.go:97-104`).
16. D3: the `run_settings` bound on `scope: off`. Held.
17. D12: the `defer` bound. Held.
18. D5: the open-range bound. Held.

**Findings**

- **F1. BLOCKING, design. D7, D8 (Bound) and Goal.**
  The baseline capture is not protected, so containment by code can be bypassed by the session it checks.
  - The baseline is `.lym/drift/baseline` (`internal/driftengine/paths.go:53,62`), and `.lym/` is git-ignored (`.gitignore:19`).
  - D8 hands the writer the exact out-of-expectation cells: case, identity, old value, new value.
  - Counterexample: the darn writer edits `.lym/drift/baseline/&lt;case&gt;` so the old value equals the new one. Its `drift` entry passes. Drift-Verify diffs against the same edited file and passes too. The verdict shows a clean check.
  - Nothing records the baseline's content at Drift-Await, such as a hash checked at every compare. D7 only promises that the check itself never writes the baseline.
  - Fix: record a digest of the baseline tree when Drift-Await counts the capture, check it before every diff, and give the mismatch a Stuck with a way forward.

- **F2. MEDIUM, design. D9 judge tree guard.**
  The snapshot misses ignored paths and the weft outside `_lym/drift/`.
  - "Untracked paths" from git status exclude ignored files, so a judge edit under `.lym/` (the baseline or a scratch capture) is not seen.
  - A judge edit to an uncommitted weft file outside `_lym/drift/` is not seen either. Examples are the decision record or darn record it is handed as evidence, or `_lym/plan/`.
  - So the claim "the judge edits no code and no expectation" holds only for the paths snapshotted.

- **F3. MEDIUM, design. D9 interrupted judge and stale `judgement.yaml`.**
  - D9 gives no rule for a judge session that dies or is interrupted. D6 gives the expect agent a probe/resume/respawn rule; D9 gives the judge none.
  - D9 also does not say that Drift-Verify removes or ignores a `judgement.yaml` left by an earlier spawn. That file is under `_lym/drift/` and is whitelisted by the guard.
  - The `judgement` entry checks only form: every cell id has exactly one valid ruling.
  - Counterexample: an earlier judge's file whose cell ids match the new spawn's (for example 1..n) passes the gate with no new ruling, so a stale `matches` gets stored as a tuple ruling.
  - A half-written file from a dead session is not addressed either.
  - Fix: bind ids to the spawn, or clear the file before the spawn, and state the interrupted-judge path.

- **F4. MEDIUM, design. D9 guard Stuck: no way forward, and the change persists.**
  - The guard Stuck names no way forward, so the parent is not told to revert the judge's change.
  - On resume, the next spawn's snapshot treats the change as "present before the spawn", which D9 says is not a Stuck.
  - At Drift-Recheck, which runs after the Code-Review perch, a judge's warp edit can then land without review.

- **F5. MEDIUM, consistency. D6 re-entry mark versus D14.**
  - `AgentSessionOpen` reads `runs.json` (`internal/driftengine/records.go:278-281`), and D14 removes "the run budget and `runs.json`".
  - Only Drift-Verify sets the mark today, through `ResetRuns` (`internal/driftengine/verify.go:565`). Drift-Baseline does not, so "as today's drift producers record" is false.
  - There is one mark per run, so the expect agent and the judge would share it, with no seat identity.
  - The re-entry guard therefore rests on a record the same record deletes.

- **F6. MEDIUM, design. D6 seal ordering claim and bound.**
  - The check compares the seal commit's `Warp-SHA` trailer text with the baseline head. A trailer is only message text (`internal/fabricengine/trailer.go:150`, `parseWarpSHATrailer`), so a hand-made commit can carry any value.
  - Counterexample: a run past Drift-Expect with no seal (D6's in-flight case). The writer writes `expected.yaml` covering its own drift, then runs `git commit` on the weft with `Drift-Seal: &lt;hash&gt;` and `Warp-SHA: &lt;baseline head&gt;`. That commit becomes the oldest seal and passes every check, even though it was made after the writer's warp commits.
  - The bound's "before the writer's first warp commit" is not what is checked. The guard holds only when an earlier genuine seal exists.
  - Fix options:
    - Check the seal commit's position against the warp's history, for example that no warp commit after the baseline head predates it.
    - Or make the seal-free past-Drift-Expect path ignore any later-made seal, for example by recording at Drift-Expect that no seal was made.

- **F7. LOW, design. D6 seal when the warp has moved.**
  - Drift-Expect seals even if the expect agent committed on the warp, so the warp head no longer equals the baseline head.
  - The mismatch surfaces only at Drift-Verify, after the writer has run, and its way forward is `--discard`, which empties the expectation. The expect agent's code commit also stays in the branch.
  - Fix: Drift-Expect refuses to seal when the warp head is not the baseline head.

- **F8. MEDIUM, design. D2 `command` mode: a `does not match` is undone by a plain resume.**
  - No ruling is stored, and D4 makes a resumed Stuck at an unchanged head check again.
  - So `lym loom resume` after a judge's `does not match` reruns the compare and spawns a fresh judge, until one rules `matches`.
  - D2's bound says stored rulings are given up, but not that a regression ruling can be re-rolled with no code change, and it names nothing that bounds this.

- **F9. MEDIUM, design. D7 lock order between the capture lock and the gate slot.**
  - A check takes the capture lock and a gate slot, and inherits a held slot when inside a gate. D7 fixes no acquisition order.
  - Counterexample with a full pool (say size 1):
    - Merriam's run-level gate holds the slot and its `drift` entry waits on the capture lock.
    - A Webster fork's `lym drift check` holds the capture lock and waits on a slot from the pool.
    - Neither proceeds: deadlock.
  - The claim "the lock serialises concurrent checks" breaks into a deadlock.
  - Fix: state the order, for example slot first then lock, or run the lock-holder without a pool slot.

- **F10. LOW, design. D1 rationale on repeated keys.**
  - An entry with `n` covers whichever line holds occurrence `n` after a shift.
  - Inserting an earlier occurrence makes the old `n=2` entry cover a different line, so the judge rules that cell against a `why` written for another line. Cells not covered become false unexpected Stucks.
  - So the claim "lets no regression through" depends on the judge catching a mismatched `why`.

**Tool calls**

1. `grep` of `internal/fabricengine/trailer.go` and `internal/driftengine/paths.go`: to check the `Warp-SHA` claim and where the baseline lives. The manifest held neither.
2. `grep` of `.gitignore` and the drift tree-snapshot code: to confirm `.lym/` is ignored. The manifest lacked it.
