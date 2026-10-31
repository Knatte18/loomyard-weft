```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the affected-set searches for F1 and F3 and check the fix covers every member.

**F1 (callers of the skip):** `positionedTurnStarts` is called from `wait.go:946` and `carryover.go:81`. `runTurnStarts` is called from `hold.go:111` and `carryover.go:101`. `operatorInstructions` is called from `carryover.go:163`. The fix documents the skip on `runTurnStarts` and `operatorInstructions`, which are the two functions that inherit it. `hold.go:111` reaches it through `runTurnStarts`, so it is covered. No member is missed.

**F3 (references to the carry-over and sent-texts rules in `doc.go`):** `doc.go` has a carry-over paragraph at lines 459-460. The reviewer said it was absent, so its premise looks false, but the fixed sentence at line 109 no longer points at it, so the fix holds either way.

Findings:
- LOW, `doc.go:460`: this sentence is the carry-over rule that `doc.go:109` restates. It says "without the first, which is the run's prompt", but `runTurnStarts` (`carryover.go:65`) now picks the prompt by `PromptLine` when one is recorded. The line was not part of this change set, and the review did not name it, so I report it as a sibling note only. It does not block the fix.

F2 had no set premise, so I left it unchecked.

Tool calls:
1. `grep` for `positionedTurnStarts`, `runTurnStarts(` and `operatorInstructions(` across `internal`, to list the F1 callers. My context did not hold the caller sites.
2. `grep` for "carry-over" and "sent-texts" in `doc.go`, to check F3's references. My context only had `doc.go` lines 95-124.
