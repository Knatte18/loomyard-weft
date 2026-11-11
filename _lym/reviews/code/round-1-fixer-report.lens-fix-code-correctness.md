Task: check the fix's code changes for correctness (F8's reordering in `AnswerParentAsk` and the added test rows).

No findings.

- **`AnswerParentAsk` write failure.** `state.UpdateJSON` returns the mutate error unwrapped and writes only when mutate returns nil. On the past-deadline path a failed write now returns the wrapped error instead of the refusal, which is the F8 fix.
- **Unchanged refusals.** The no-ask, not-latest and ended refusals return `errParentAskUnchanged`, which the new `errors.Is` check skips. They still reach the `refusal` return. A failed report write returns a non-unchanged error and is wrapped. The success path is unchanged.
- **F8 has no regression test.** The review asked for none, and a write failure is hard to induce, so this is a gap and not a defect.
- **Unknown-state rows (F5).** With `Ended()` changed to an explicit list of states, `future-state` would be answered or expired, so both new rows fail with the defect in place.
- **Report-findings assertion (F2).** A dropped severity, class or id mapping in `readParentFindings` fails the new `wantReport` check. The first finding must carry MEDIUM and design, and the second must carry neither.
- **Caller-guard rows (F3).** "The run's parent orch answers" uses `strandName` `ly:orch`, which differs from the ask's strand `ly:orch-asked`, so it reaches the `parent.Name` branch. The unresolved-parent row reaches the `parentErr` branch and the "could not be resolved" text.
- **Findings-file rows (F4).** The `emptyFindingsFile` flag writes the empty file, and the pre-set nonexistent path is not overwritten because both overwrite conditions are false. The `"["` and whitespace-change rows reach the refusal branches they name.
- **Cut-short wait (F6).** `sleepErr` returns before the clock advances. The `end` deadline is a minute away and the ask's deadline an hour away, so the loop reaches the sleep and refuses with "the wait was cut short".
- **Callers.** `wantErr` changed from `string` to `[]string`, and every table row and use site is updated. `wantFindings` was replaced by `len(wantReport)`, with no remaining reference.
- **Gate.** The tier0 gate passed on the changed packages.

Tool calls:
1. Read `internal/state/state.go` `UpdateJSON` by grep, because my context lacked whether it propagates or swallows the mutate error and whether it writes after one.
