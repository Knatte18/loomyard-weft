```yaml
lens: fix-way-forward
outcome: reported
```

fix-way-forward: checking that every refusal, step and test the fix introduced can be carried out and makes progress.

**1. MEDIUM — D13 seam refusal and D1 guard on a corrupt state file**
- Changed text: D13, "When both hold begun batches, or either state file fails to load, the seam refuses, naming the file and D1's reset verbs as the way forward." D1, "a corrupt one refuses, naming the file".
- Problem: for a corrupt Webster state file, the named reset verb cannot run.
- Evidence: `lym webster reset` loads the state first and fails on a load error (`internal/webstercli/reset.go:95-98`: `st, err := websterengine.LoadState(...)` then `return fail(err.Error())`).
- D1's corrupt-file refusal names only the file, with no step to take.
- Either refusal needs a step that works from a corrupt file, such as moving the named file aside or a reset mode that tolerates an unloadable state.

**2. MEDIUM — D1 names `lym webster reset` with no target**
- Changed text: D1, "with `lym webster reset` for Webster's and `lym weaver reset` (D2) for Weaver's". The AC adds "a test runs each named step and the row then proceeds".
- Problem: the command as named cannot be run, and the D1 refusal does not say which target clears Webster's begun batches.
- Evidence: `lym webster reset` requires `--to &lt;target&gt;`, one of start, pre-fix, report-head, last-batch-head or batch-start (`internal/websterengine/reset.go:23-39`).
- Only a reset to start archives the run record (`ArchiveRunAfterReset`, `internal/websterengine/archive.go:160`), and that reset moves the task branch back to the run's start.
- That reset can itself refuse over pending audit findings, each refusal naming its clearing step.
- The refusal should name `lym webster reset --to start`.

**3. MEDIUM — `lym weaver reset` leaves Weaver's built commits on the branch**
- Changed text: D2, "`lym weaver reset`, which archives `_lym/weaver/` and clears `.lym/weaver/`, and touches neither the plan, the branch's commits nor Webster's run directory". The D1 way forward is "reset the other implementer's run directory".
- Problem: after this reset, Webster starts with every card un-done while Weaver's batch commits stay on the task branch.
- Webster's batches then rebuild work that has already landed. D1's "A run never switches implementer with work half built" is not met, because the built work remains.
- The asymmetry: the Webster direction can return the branch to the run's start, and the Weaver direction cannot.
- Evidence: Webster's reset to start moves the branch and archives the run (`reset.go:190`, `archive.go:160`).
- The fix should either give `lym weaver reset` a move back to the run's start commit, which D3 already records per batch, or state that the operator reverts Weaver's commits as part of the step.

**4. LOW — D2's run-lock refusal**
- Changed text: D2, "refuses while the run's shed status is `running` or Weaver's run lock is held, naming `lym loom pause --after` the implementer row".
- Problem: when the lock is held by a crashed producer, pausing after the row does not release it, so the named step does not lead out.
- D12 only says the producer holds the lock for its whole call.
- Whether a stale lock frees itself, through the `lock` package's holder-liveness check, is not stated. The refusal's way forward fits only the `running` case.

Tool calls beyond context:
- grep of `internal/websterengine/reset.go` targets and LoadState: to learn which `lym webster reset` targets exist and whether it loads state.
- sed of `archive.go` and `reset.go` spans: to see which target archives the run record.
- grep for PlanReset callers: to find the reset verb's CLI file.
- sed of `internal/webstercli/reset.go:88-105`: to confirm the verb fails on a state load error.
