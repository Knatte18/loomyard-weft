```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F12 change set for sibling text or twin code left out of step.

**Findings**

1. LOW — `internal/weaverengine/run.go:97` (`RunActive`), left behind by the F7 change in `reset.go`.
   - `Reset` now takes the lock itself through `lock.TryAcquireWriteLock`, so `weaverengine.RunActive` has no production caller.
   - My grep of `internal` and `cmd` finds only its definition, plus test uses in `reset_test.go` and `run_test.go`.
   - The `RunActive` that `webstercli/ownership.go` and `websterengine/reset.go` call is Webster's own.
   - It is now dead exported code kept alive by tests. Either delete it and probe through `Reset` or `Run` in the tests, or confirm a CLI caller is coming.

2. LOW — `internal/weavercli/reset.go:29-37`, the `lym weaver reset` help text, which F2 left stale.
   - Line 31-32 gives "commit it, or restore it with `git checkout -- <path>`" as the way out of a tracked-path refusal.
   - It does not say that `--stop-strand` now counts every tracked path dirty after the stop as the run's own and discards it. Line 22-24 mentions only "the uncommitted changes the run left".
   - That is a new data-loss path: a parent's concurrent edits are discarded along with the stopped strand's.
   - The `Reset` doc comment in `reset.go` states it ("The uncommitted edits of a strand it stopped count as the run's own"), and `weaverengine/doc.go:80-81` lists the refusals without it. Both CLI docs lack the matching sentence.

**Checked and consistent**
- F9 (`run.go` `beginBatch` doc, `state.go` `StopRecord` comment): they agree with `recordDoneWithoutSpawn`, `checkRecordable` and `finishRun`, which are the only places that clear the stop record.
- F8 (`nextAttemptRung` doc, `judgeOpenBatch` doc): they agree.
- F6 (`truncateCause`, `causeWayForwardMarker`): `stuckReasonBytes` has no other user.
- `contracts/specs/implementer-spec.md` carries no wording about refusal order, fault retry or the stop record.
- F10 (`PATTERN-cwd-resolution.md`): the list is consistent.

**Tool calls**
1. `grep` for `stuckReasonBytes`, `RunActive(` and the implementer-spec wording, because my context did not show other users of the changed symbols.
2. `grep` over the `weavercli` and `weaverengine` docs and the `stopKindBatch`/`outcomeFault` twins, because I needed to know whether other docs restate the refusal order or the stop kinds.
3. `Read` of `internal/weavercli/reset.go:20-49`, because my context lacked the verb's help text.
