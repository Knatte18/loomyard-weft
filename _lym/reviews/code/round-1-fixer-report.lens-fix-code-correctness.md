```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the fix through the correctness lens, running the added and changed code and tests against concrete inputs.

No findings.

What I traced:
- **F1, fold row:** the loop sets `StartedAt = "s"+At`, so the first call is `st1`. The test asserts `st1`. A fold that overwrote `StartedAt` would give `st2` or `st3` and fail the row.
- **F1, `assertCallTimed`:** an empty `StartedAt` fails `assertRFC3339UTC`. A value later than `At` fails the comparison. Both RFC 3339 UTC strings compare correctly as plain strings.
- **F1, `StuckWithinBudget` and `StuckWithNoOnStuck`:** they index `res.History[0]` only after `len == 1`. The seed has an empty history, and the gate ran green.
- **F1, reroute and capped entries:** neither can fold, since `foldsInto` excludes `RerouteCapped` and the reroute entry is not budget-exempt. The `last` entry is therefore the one that was just written.
- **F3, capped row:** with the `capped` branch of `abandonLoad` removed, `APIError` would be `""` and the test would fail. The last turn's text is distinct, so the test checks the newest error text and not just any error.
- **F3, all-outputs-exist row:** with the `classifyDeadlineExpiry` return removed, the outcome would be `api-error`, not `done`, and the row would fail. Writing the output file in `PrepareHook` is correct, because the spec refuses a file that exists at start.
- **F3, restructured `cappedHold` loop:** it now runs `loadHoldMaxAPIErrors-1` iterations, adding the same back-offs as the old `if i < max-1` loop. `wantFailedAfter` is unchanged.
- **F4:** `notices[0]` is indexed only after `len(notices) == 0` is ruled out. If the notice text lost its cap wording, the substring check would fail.
- **F5:** the `idle_gaps[0]` map is compared with `reflect.DeepEqual`. The JSON tags and the `10m0s` idle value are pinned, so a tag rename would fail.
- **F2 and F6:** comment and doc changes only. The folded-entry statement matches `idleGapsOf`, which measures from `LastAt` and never between folded calls.

Outside my lens: the `rerouteStuck` path at `reroute.go:49` (the Stuck entry written when a reroute has no `OnReroute` target) gets no `StartedAt` assertion from the new test lines. I did not verify whether an existing test reaches it. It is not a defect in the change.

Tool calls made: none. My context already held every changed file, and the gate's green output for the three packages.
