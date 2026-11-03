```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix-code-correctness lens over the changed code in `dependency.go` and the tests added for F1–F4.

No findings.

Traces I ran:
- **`metBelow`, two projects each naming the other.** `loop-a` is marked as judging, then `loop-b`, then `loop-a` again, which `judging[slug]` rejects as not met. So `loop-b` and `loop-a` are both unmet and `Pending` returns the dependency.
- **`metBelow`, a project naming itself.** Its only member is itself, which hits the guard, so it is unmet.
- **`metBelow`, a diamond of shared members.** `defer delete(judging, slug)` clears each slug on exit, so a member reached by two paths is not mistaken for a loop. Non-loop boards judge as before.
- **`standsForBelow`, loop case.** For `loop-a` the result is `[loop-a, loop-b]`. `loop-a` is already listed on the path when it comes up again, so it returns nil and the walk ends. The self-naming case returns `[self]`. Without a loop the result is the same as before.
- **`ComputeLayers`.** `detectCycleDFS` calls `met`, which now terminates, and it reaches `gray` on `loop-a` and returns the cycle error before `getDepth` runs. `getDepth` follows the same unmet edges, so it is never reached with a loop and needs no guard of its own. Leaving `layer.go` unchanged is sound.
- **Callers.** `met` and `standsFor` keep their signatures, so `Pending`, `overlap`, `afterCell` and the siblings filter hold.

Whether each new test fails with its defect in place:
- **Pending rows `loop-a` and `self-member`.** Without the guard the recursion overflows the stack, so they fail.
- **`TestComputeLayers` loop rows.** The same overflow happens in `detectCycleDFS`, so they fail.
- **README row.** `after-finished` would show `P2` under After if `afterCell` fell back to `isDone`, because `finished` is not marked done. It would then fail against the golden.
- **Siblings row.** With `isDone` in the filter, `P` would be added as a dependency and the `X`/`Y` sibling would be reported. The row expects none, so it fails.
- **Gate.** All of these passed in the fixer's gate run, which reported 651 passed and 0 failed.

`standsFor` termination on a loop has no direct test, because it is unexported and a loop cannot be written through the Board. The review did not ask for one, so I am not raising it.

Further tool calls: none. The context already held every changed file and the gate result.
