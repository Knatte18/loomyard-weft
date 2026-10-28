```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F3, F4, F2/F5), looking for a case where each is wrong.

**Findings**

1. **LOW**: F1, `activity.go` in `ReadAgentActivity`, `reading.WaiterGone = true`.
   - The proposal sets `WaiterGone` for every non-live running record, including a pid-less one (`PID <= 0`, from an older binary).
   - The `WaiterGone` doc says "the process recorded as waiting on it is gone". A pid-less record records no waiter.
   - `noteWaiterlessRun` deliberately does not count a pid-less record as waiterless ("A record with no pid predates the field and is not counted"). `WaiterGone` now disagrees with it.
   - The `activity_test.go` row for `run-b` pins `WaiterGone: true` for the pid-less run.
   - The observable result is consistent: `runIsLive` already reads pid-less as not live and zeroes the API error and usage limit.
   - But the field's name and doc claim more than the code knows. A reader who relies on the doc would be wrong for pid-less records.

2. **LOW**: F3, `signals.go` in `typedInstruction`, "first word starts with `/` and holds no further `/`".
   - A single-segment path opening an instruction, such as "/etc is unreadable, fix it", is still read as a slash command and dropped on carry-over.
   - The reviewer's rule cannot tell `/etc` from `/compact`. The gap is narrower than before but real, and no test row documents it.

**Challenged and held**

- F4: carrying a first turn start whose instruction differs from `PromptLine` cannot carry the task prompt as an operator instruction when the paste text and the hook prompt differ. `PromptLine` goes out through `typeVerified`, which calls `recordSentText` (`run.go:1605`). `sentTexts` therefore already filters it, so the proposal has a second guard.
- F2/F5: the added rows and the retry test do fail with the defect in place. Only the `PromptOffset` and `PromptLine` seeding is shared, and no case I could build makes a row pass vacuously.

**Tool calls**
1. `grep -n "PromptLine" *.go` plus `sed -n 1595,1610p run.go` in `internal/shuttleengine`: my context did not show where `PromptLine` is typed or whether that path records the sent text.
