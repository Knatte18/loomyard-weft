---
round: 2
focus:
  - "internal/shuttleengine/startup_test.go `TestStartup_SendFailuresTearDown` and internal/shuttleengine/run_test.go `TestRun_Send_ConfirmsSubmission`: does each test, with its subtests, call `t.Parallel`, or carry a comment at the test or once atop its file naming the process-global state that excuses running serially (PATTERN-test-speed)?"
  - "internal/shuttleengine/wait.go `maxUnsentReprompts` and the gate loop that ends on it: does the plan's Card 6 bound (re-prompt retry spends no gate budget, bounded by the run deadline) allow ending a gate early after consecutive busy or unlanded re-prompts, and if not, does the diff match the plan and the decision record's D6?"
  - "internal/shuttleengine/send.go file header against the file's contents (settle, confirm and clear path, `ErrSubmissionNotLanded`, `Run.sendWithin`, `Run.boundaryIdle`, `Run.startHoldReleased`): does the header describe what the file holds, under the repo's comment conventions?"
---

The entries name sites and questions only; each is to be settled against the current code and the plan.
