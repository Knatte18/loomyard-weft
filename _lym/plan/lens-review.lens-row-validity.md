```yaml
lens: row-validity
outcome: reported
```

Findings (row-validity):

1. **MEDIUM — Card 4, `TestSamplerSession`.** The rows that keep waiting have no stated way to end the call, and the card does not list the helper they need. Affected rows: a flag read on a dirty checkout ("clears the flag, leaves the session live and mails it once"), and a covered wake followed by an uncovered resume. In both the producer loops forever: the wait has no timeout, and a cleared flag or a resumed session never returns a verdict. The card lists only `fakeSamplerSessions`. It names no scripted `Sleep` or `PauseCovered`, and no cancel-after-n-wakes helper like `waitWorld.cancelAfter` in `internal/landingshed/prreview_wait_test.go:103`. Without one these rows hang. With a cancel, the outcome is a cancelled-context error, and the card should state that as the row's end. Evidence: card 4 Intent lines 39–43, and `internal/landingshed#PRReview.wait` loops until a verdict.

2. **MEDIUM — Card 4, the `Watch` row.** The row needs a fake `Sleep` that blocks until its context is cancelled, and the card does not say so. In `BeamAwait.pause` (`internal/pawlengine/beamawait.go:332-349`), `Sleep` runs in a goroutine and is cancelled when a `Watch` event arrives. A fake `Sleep` that returns at once makes the select race, so "the fake clock showing under a second of sleep" may hold whether or not the watch cut the sleep short. The row would then not fail when the code is wrong. Evidence: card 4 line 43.

3. **MEDIUM — Card 12, `agentActivityFor`.** The new status read has an absent-or-unreadable branch ("reads as before, with no error"), and no planned row reaches it. `TestAgentActivity_LiveLockWaitReadsAsActivity` gains only well-formed status rows. Evidence: card 12 lines 29 and 33. The existing table, `internal/loomcli/reviewstatus_test.go:375`, has no malformed-status row.

4. **LOW — Card 6, `TestDarnVerifyBudget` sampler row.** The row says "`New` with `DarnVerifyAttempts` 7 stamps 7 on Code-Burler-Fix's verify gate". The built `Shed` gives no way to read a producer's gate attempts. The existing subtest asserts the stamp on the parsed recipe through `verifyGatedRows` and `applyVerifyAttempts`, not through `New` (`internal/loomrecipe/darnrecipe_test.go:276-305`). As written, the row's assertion cannot be made. It should check the parsed recipe as the darn subtest does.

5. **LOW — Card 8, `TestSamplerRecipe` row.** The row says "`CodePerchGate(sampler, …)` is the perch rows' gate". Go func values cannot be compared, so the row has to evaluate the gate: false with `code_review` false, true with true. Otherwise its assertion is not defined. Evidence: card 8 line 41.

6. **LOW — Card 5, `TestSamplerSessionEntry` repeats coverage.** Its "a filled env builds a producer" and "a `bogus` config key is refused" rows repeat what `TestEveryEntry_RejectsUnknownConfigKey` already checks once `minimalEntries` gains the `SamplerSession` row. That test builds the minimal entry and rejects `bogus_key` for every registry name (`internal/shedrecipe/conformance_test.go:104-127`). Only the nil-seam rows are new.

7. **LOW — Card 11, `TestDecisionVerb_Refusals` repeats coverage.**
   - "Sampler strand with `--by arbiter` refused by the arbiter rule" repeats the existing "arbiter label from another task strand" row, which takes the same `arbiterStrand` false branch (`internal/loomcli/decision_test.go:617`).
   - "A task strand the reader does not admit, refused with the parent way forward" repeats "named caller differing from the recorded parent" (`decision_test.go:575`).
   - The non-admission cases are new only in `TestSamplerStrandReader`.

8. **LOW — Card 7, `TestNewTask` repeats coverage.** The new "project sampler" row reaches the same `checkProjectFields` branch as the existing "project tapestry" row, which already pins the refusal for any recipe other than pawl (`internal/boardengine/task_test.go:187`).

9. **LOW — Card 7, `TestPrepareRunSettings` repeats coverage.** The row "recipe `sampler` with no profile writes no document" takes the same branch as the existing "no profile and no default for the recipe writes the task's settings alone" (`internal/battencli/runsettings_test.go:32`). The fact that sampler has no default is pinned already by card 7's `TestTemplateLadder` assertion that `Default("sampler")` is empty.

10. **LOW — Card 10, `TestSamplerDoneVerb` repeats coverage.** The row "also when the run is halted at the row" gives the verb exactly what the success row gives it. The fake `currentRow` returns only a producer name and the verb reads no state, so "halted" cannot be told apart from running. The row adds nothing (card 10 lines 27 and 39).

11. **LOW — Card 3, `TestSessionSpec`.** The row asserts the stencil's static prose: that it names `lym loom sampler-done`, `--by operator`, `tmux -L`, the sandbox hub, and holds no `git push` or `gh pr`. The Plan-Review rubric's "Test economy" flags assertions on static prose. The decision record's D8 and D9 acceptance criteria ask for exactly these checks, so the conflict should be recorded rather than the row dropped.

Tool calls:
- One grep over `internal/boardengine/task_test.go` project rows, the `TestPrepareRunSettings` table and the `TestTacklerFor` table. My context did not hold those test tables, and the checks for repeated rows and reachability needed them.
