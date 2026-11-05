```yaml
lens: fix-proposal
outcome: reported
```

Lens fix-proposal: I challenged the proposals the fixer carried out as proposed, F2, F3 and F4. F1 went to the parent.

Findings:

1. **MEDIUM — F4's `deps.go` FileTask error text.** The proposal leaves the rerun's way forward unchanged, so the problem the finding described still happens.
   - After a failed release, the draft task stays on the board. `openMergeTask` (`mergeparent.go:94-101`) only skips tasks with a done status, so it finds the draft. The next `merge-parent` run still refuses at `mergeparent.go:50-51` with "stands open … `lym batten cancel <task> --remove`" for a task that never had a run.
   - The first failure is also wrapped at `mergeparent.go:83` with "way forward: fix the cause it names, then run `lym beam merge-parent` again". Following that advice ends in the second, wrong refusal.
   - The F4 text itself, "remove it with `lym board` or release it, then retry", names no concrete command, and the retry advice contradicts the rerun's refusal.
   - A sound change would make the "stands open" refusal tell a draft task apart (`IsDraft`, `draft.go:15`), or the first error would not append the generic "run again" advice.

2. **LOW — F2's `CodeReviewCaps` assertion.** It cannot catch a wrong perch or wrong config source.
   - The test compares `deps.CodeReviewCaps()` with `cfg.Caps(loomengine.PerchCode)`, loaded the same way from the same default hub config.
   - `Caps` (`caps.go:35-46`) falls back to the shared `ReviewMaxBounces` unless a per-perch value is set. In a default hub the plan, discussion and code perches likely resolve identically, so a mutation to `PerchPlan` would still pass.
   - The review asked for "the caps the hub's loom config sets for the code perch". The test never sets a code-perch-specific value in the hub's loom config, so it cannot show that the code perch is the one read.

F3 holds up: `IsDoneStatus` is exactly `status != nil && *status == StatusDone`, so behaviour is unchanged. I found no breakage in the test's `done := boardengine.StatusDone`.

Tool calls: none. My context already held every changed file.
