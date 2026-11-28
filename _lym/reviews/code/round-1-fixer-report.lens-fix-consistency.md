```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the F1–F3 change set agrees with its surroundings.

1. **MEDIUM**: `_lym/landing/summary.md`, `## What changes`, the unchanged bullet that F3 made stale.
   - The bullet says "`TestTemplateLadder` now pins that no starting profile holds an arras block and that arras has no default."
   - F3 (commit `0b3e76ed8`) deleted the `Default("arras")` assertion and cut "arras has no default profile" from that test's doc comment, so the test no longer pins the second half. `TestLoad`'s template row does.
   - The same section also leaves out the new test `TestReconcileHubWideAt_ProfilesStillLoadAfterTheArrasBlocksDrop`, which is a smaller gap.

2. **LOW**: `internal/runprofile/reinterpret.go`, `ReinterpretArrasDefault`, against the stated reinterpretation contract.
   - `internal/configreg/doc.go` says a reinterpretation is "a rewrite that keeps what the hub meant, or flags a value it cannot settle with a way forward".
   - `pattern/PATTERN-config-layers.md`, "Config reinterpretation", says it "rewrites the value to its old meaning, or flags it with a way forward".
   - The new rewrite does neither as worded. It drops the operator's default, so the meaning changes ("now runs on the hub's own config"), and it reports this in `rewrote`. Its doc comment says "nothing is ever flagged", and the new test fails on any flagged line.
   - The line does carry a way forward and reaches the orch like a flagged one. The least change is to return it as `flagged`, which is the contract's route for a value whose meaning cannot be kept. The test's `flagged` and `rewrote` expectations would move with it.

I found no other stale twin: `internal/runprofile/doc.go`, the `configreg.go` registration and the F3 doc comment agree with the code.

Tool calls: none.
