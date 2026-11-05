```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the proposals the fixer carried out as proposed (F1–F4).

**Findings**

1. **LOW** — `shortnamebinding_clone_integration_test.go`, new comment above `SecondCloneDerivesShortname`: "they all read the one weft fixture and existing hub (firstParent and first)".
   - The claim is wrong for the first two subtests. `SecondCloneDerivesShortname` and `DisagreeingShortnameRefuses` use only `weftURL` and a fresh `t.TempDir()`. Neither touches `firstParent` or `first`.
   - `ResetKeepsRecord` tears down only the hub, never the weft remote, so it gives those two no ordering reason.
   - The proposal's own alternative (`t.Parallel` on the two read-only subtests) was the sounder fix. The comment now states a shared-hub reason that holds only for the last two subtests, so a reader is misled about why the first two are serial.
   - Suggested rewording: say that only `ResetWithDisagreeingShortnameKeepsHub` and `ResetKeepsRecord` share the hub, and either parallelise the other two or state their own reason.

2. **No finding on F1, F2 or F4.**
   - **F1:** the rewritten summary matches the diff. The `--subpath` check runs before the reset, and the test list matches what the files contain.
   - **F2:** each row fails with its defect in place.
     - Dropping the `FETCH_HEAD` read on row 1 gives the "does not exist as a directory" refusal instead of the "recorded anchor" one.
     - Dropping the fallback on row 2 does the same.
     - Dropping the default-branch gate on row 3 adopts `elsewhere` and refuses `backend`.
   - **F4:** `checkResetTarget` is in `clone.go:656`, so the attribution is correct.

Further tool calls:
- `grep -n "func checkResetTarget\|func resetHub" internal/fabricengine/*.go`: to confirm `checkResetTarget` lives in `clone.go`, since my context didn't show where it is defined.
