```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens, tracing each change in the diff to the goal points of the board entry.

**Per change**
- C1, `internal/battencli/lifecycle_integration_test.go:336-425`: the flat step table becomes a table of groups. Each group calls `t.Parallel`, gets its own `hubforge.CopyHub`, and runs its steps in order. Traces to Work 1, "Split it into groups with a hub each, so the groups run in parallel".
- C2, `internal/battencli/lifecycle_integration_test.go:342-348`: the rewritten doc comment of `TestBattenIntegration_Rows` and the "last in its group" comment at :399. Forced by C1, because the old text described one shared, serial hub.
- C3, `pattern/PATTERN-pair-remote.md:140`: the subtest path changes to `TestBattenIntegration_Rows/CreateRow/RecordsPrimeRefusal`. Forced by C1, because the group level changes the subtest path.

**Per decision** (goal points under `## Goal`; the record has no `Added after Discussion` entries)
- Work 1, split into groups with a hub each, run in parallel: built by C1, with C2 and C3 forced.
- Work 2, keep every assertion and drop none: built by C1. The old table held 36 steps, and the new groups hold 6 + 1 + 3 + 6 + 7 + 7 + 6 = 36. Each old step appears exactly once, and no step function body changed (the diff touches only `lifecycle_integration_test.go:336-425`).
- Work 3, leave the cancel tests' structure for a later task: met by leaving them alone. No cancel test file is in the diff, which holds only `internal/battencli/lifecycle_integration_test.go` and `pattern/PATTERN-pair-remote.md`.
- Work 4, measure the package before and after three times each and put the figures in the change description: built outside the committed diff, in `_lym/landing/summary.md` under `## Measurements`, with three before runs and three after runs. The handler owns whether those figures match the Aim's setup (the focus entry on Measurements); this lens only confirms the measurement exists.

**Findings**
None. Every change traces to a goal point or is forced by one, every goal point is built, and no commit carries a `Pattern-Departure:` trailer (`git log` over the range shows commit `d9a84eb2` alone, with no trailers).

Tool calls: none; my inherited context already held the diff, the decision record, the board entry, the summary and the commit log.
