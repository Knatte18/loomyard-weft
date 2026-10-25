```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix set for consistency with its surroundings (fix-consistency lens).

**Finding (LOW):** `internal/dobbyengine/doc.go:36`, the "continue pass" bullet, is now stale.
- It says "A continue pass needs the run at Run-Shed with no full pass owed".
- The fix to `planPass` adds a third condition: a continue pass also needs no lift to answer.
- The `planPass` doc comment in `passes.go` was updated to say this. The package doc, its twin, was not.
- The package doc already says (line 38) that the full pass runs at the lift, but it never ties that to the uncovered lift case.
- Suggested edit: add "and no lift to answer" to the `doc.go` bullet.

No other inconsistency found:
- `boardengine/doc.go:156` ("the next write sets `draft`… in the same save") still holds with the earlier migration.
- The reworded `boardCriticalSection` doc comment matches the new step order.
- The `battenshed` comment reword has no twin elsewhere: `grep` for "loom config" found only test-file hits.
- `doc.go:35` ("a lift while a pass is in flight…") is consistent with the new behaviour.

Tool calls:
1. `grep` for "lift" in `dobbyengine/doc.go` and "migrat" in the board docs: to check the docs for stale statements, which my context lacked.
2. `sed -n 30,42p internal/dobbyengine/doc.go`: to read the trigger and continue-pass bullets in full, which my context lacked.
