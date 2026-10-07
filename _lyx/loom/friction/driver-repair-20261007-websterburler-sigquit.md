# Driver repair: interrupted Webster-Burler step

```yaml
run_id: run-recovery
key: Webster-Burler / history_length 26
repair_count: 1 of 2
```

## Failure

`lyx shed step run-recovery` (trace `b8a016cec751fbbb`) exited with code 2 on SIGQUIT and printed a goroutine dump instead of an envelope.
The dump shows the step inside `burlerengine.(*Engine).Run` → `shuttleengine.(*Run).Wait`, so it was waiting on the round-2 Burler agent.
Status afterwards: `current_producer` Webster-Burler, `history_length` 26, `state` running, `last_step.finished` false, `binary_changed` true.

## Trace lines acted on

```
shed: step trace=b8a016cec751fbbb
burler: round starting round=2 reviewPath=.../_lyx/reviews/webster/round-2-review.md
shuttle: run started runDir=.../.lyx/shuttle/b25d04dbbd4a6c4ae357a2c00bc247ac strandGUID=7bf79ebd6bc17cf903c16ad552ac6af6
```

No `fabric: mutation` record follows, so no half-finished mutation exists.

## Action and outcome

The run is unchanged and `interrupt_policy` is `reinvoke`, so the step is invoked again.
Outcome: see the stop report for this attempt.
