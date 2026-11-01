```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked whether each refusal, error, next step and criterion the fix added can be carried out from the state it names, and whether it leads somewhere useful.

Ways forward checked:
- **D8 refusal of a halted run or one moved by goto:** `lym loom goto --to Parts-Plan-Write`, then `lym loom resume`. This can be carried out. `Goto` refuses only a running run or a done one. It accepts a blocked, failed, paused or awaiting run, and it can target the current row because `gotoAdmitted` works from the current row's index. Goto leaves the run paused, `lym loom resume` exists, and the chair is not live, so the fresh spawn runs (D6 Bound). The way forward makes progress.
- **D8 refusal of a caller that is not the chair:** a part writer, orch or the operator is sent to the chair or to `insert-card` once the run is past the row. This can be reached.
- **D9 open fix after a resume:** the chair fixes the part itself, then runs `join-parts`. The chair can check its fix with `validate-plan --part`, and `join-parts` reruns the part check. This can be reached.
- **D6 writer timeout → Stuck with a committed pointer:** the way back is a rerun of the halted row (D1), which archives the parts. This can be reached.
- **D10 `lym loom validate-parts` before the turn ends:** the tapestry recipe holds the `Parts-Write` row whatever `plan_split` is set to, so the verb is admitted during that row. This can be reached.
- **The new acceptance criteria for D2, D3, D4, D5, D6 and D8:** each names a state a test can build.

Findings:

1. **LOW.** Changed text: D3, "its refusal's way forward says the verb serves only a run of such a recipe, whose `Parts-Write` agent runs it as its self-check."
   Summary: the refusal explains why the verb does not apply but names no action, so it does not meet PATTERN-refusal-way-forward's "names its way forward". The record lists `validate-parts` under that constraint.
   Evidence: every other refusal in the record names a command or edit, for example D8's "`lym loom goto --to Parts-Plan-Write`, then `lym loom resume`". This one names none. It could say the verb has nothing to check in this run and that the plan is checked with `lym loom validate-plan`.

2. **LOW.** Changed text: D8, "a halted run or one moved by goto: `lym loom goto --to Parts-Plan-Write`, then `lym loom resume`".
   Summary: for a run already halted at `Parts-Plan-Write`, or already moved there by goto, the goto step does nothing more than leave the run paused. The resume alone reruns the row, as D1 says ("the operator's way back is a rerun of the halted row").
   Evidence: `internal/shedengine/goto.go:118-124` sets `CurrentProducer` to the target and the state to paused, which for this case changes nothing that resume needs. The way forward still works, so this is only redundancy a reader would notice.

Tool calls:
- grep of `internal/shedengine/goto*.go` for state guards: my context did not hold goto's rules on which states and targets it accepts.
- sed of `internal/shedengine/goto.go` lines 95-130, 145-170 and 255-265: the running/done refusals, `CheckGotoTarget` and `resumableHalt`, to confirm goto accepts the current row on a halted run.
- grep of `gotoAdmitted` and the `resume` verb: to confirm goto can target the current row and that `lym loom resume` exists.
