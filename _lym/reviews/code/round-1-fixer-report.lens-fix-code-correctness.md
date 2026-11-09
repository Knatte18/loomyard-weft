```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the code and tests the fix added or changed (fix-code-correctness lens).

I found no defects, so the report below is "no findings".

- **`verifybudget.go` (`failures`, `verifyBudgetNotice`):** `verifyBudgetNotice` has one caller, `noticeIfSpent`, which passes the slice under the lock. The function clones before it reverses, so the stored order is untouched. With two failures, the latest comes first, and the cut runs on a rune boundary as before.
- **`gateEndedEarly` (reordered in `wait.go`):** the body is unchanged, so behaviour is the same. The doc comments now sit above their own funcs.
- **`TestGate_EndedEarlyStaysEmpty`:** it fails if `EndedEarly` is set unconditionally, because the pass, terminal and spent-attempts rows all expect `""`. The spent row only reaches its second boundary if the turn-2 line is appended after the first send.
- **`EndedEarly` rows:** the three cause rows are exact enough that returning an empty string or one fixed string breaks at least one. The wait-expiry row has an empty entry name, so it matches only on `entry's wait ran out`.
- **`TestVerifyGate_NoticesTheParentOnceItsBudgetIsSpent`:** it fails if `clock.arrive`, `clock.arm` or `clock.failed` is deleted. Without `arrive` there is no notice, without `arm` `delays` stays empty, and without `failed` the notice says "no verify run has failed yet".
- **Rerun table:** the rows depend on a commit per `keepTestA`, because a recorded pass for the alone call on an unchanged tree would otherwise skip it. If the timeout or crash run kept its tests, the second `gate()` would add a `lym` call and the `callsBefore+1` assertion would fail. The dirty row checks that the tests are kept by the extra alone call after the tree is clean.
- **Rune-boundary cut test:** the two pads shift the multibyte log path by one byte, so one of them puts the 1000-byte cut inside a rune.

Minor, no action needed:
- `TestRunFailedTestsAlone_UnrenderableCallLeavesTheArrivalToTheFullCommand` reaches the render-error branch only through an empty list. `NewVerifyGate` never passes one, because it checks `len(failedBefore) > 0` first. The branch is still exercised.

Further tool calls: none, because my context already held every changed file and the gate output.
