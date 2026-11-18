```yaml
lens: fix-claims
outcome: reported
```

Task: check, through the fix-claims lens, every bound, behaviour or code fact the F13 fix added or rewrote against the code at HEAD.

no findings

Claims checked against code already in context:
- "A producer's `Call` receives no shed history": holds. `ShedProducer.Call(ctx context.Context)` at `internal/shedengine/producer.go:90` takes only a context.
- "shed then persists the reroute history entry, whose output is that file": holds. `rerouteStuck` in `internal/shedengine/reroute.go` writes `Output: output.Path` on the `OutcomeReroute` entry. `foldsInto` (`internal/shedengine/run.go:744`) never folds a reroute entry because that entry is not `BudgetExempt`, so each send-back's entry keeps its own output.
- "the resume after a reroute-cap halt, whose entry is a Stuck one": holds. The cap-halt entry is `Outcome: Stuck, RerouteCapped: true`, and `persist(st.CurrentProducer, StateBlocked, …)` leaves PR-Review current, so the resume re-calls the row.
- "reroutes that send-back within the reroutes the resume grants": holds. `rerouteCount` stops at the `RerouteCapped` entry, so the resumed row's reroute counts from zero.
- Crash before the persist re-calls the row with no reroute entry for the mark's file: holds per `internal/shedengine/run.go:405`. Matching on the file tells this apart from an earlier send-back's reroute, because each send-back's `pr-findings-<n>.md` path is distinct.
- The dirty-tree Stuck and a re-step leave no reroute entry naming the mark's file: holds. Only `rerouteStuck`'s below-cap branch writes `OutcomeReroute`.

No further tool calls made.
