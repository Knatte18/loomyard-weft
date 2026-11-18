```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for stale siblings and left-behind twins (lens fix-consistency).

**LOW — `_lym/landing/summary.md`, "A prompt line that started no turn is typed once more" and "Tests" sections.**
- The sections still describe the pre-fix retype.
- The conditions list says "no turn start of any instruction lies past the prompt offset". The code now judges from `promptSendOffset`, the events size just before the typing, in `hold.go` (`turnStartedSince(..., run.promptSendOffset)`).
- The conditions list omits the new "run deadline not passed" condition.
- "It does this at most once per handle" is no longer exact. A retype held busy before it typed leaves the retype unused (`errors.As(err, &amp;busy)` returns before `lostPromptRetyped = true`).
- The "Tests" section lists only the earlier rows. It does not mention the new `TestWait_RetypesAPromptLineThatStartedNoTurn` rows, `TestStart_RetypesAPromptLineThatStartedNoTurn`, or the new `TestStartup_SendFailuresTearDown` rows.
- The fixer updated this file only for the F3 outcome paragraph.
- The sibling docs it matches (`hold.go` doc, `doc.go`) were updated, so this is the one twin left stale.

No other inconsistencies found.
- The `retypeLostPrompt` doc, the `doc.go` retype and start paragraphs, `typePendingPrompt`, `sendStartPrompt`, `sendThroughPause`, the `promptDelivered` field comment and the `abandonStartup` doc all agree with the code.
- `deliverPromptAtHold`, `deliverPromptOnAttach` and `DeliverPendingPrompt` still send with a zero deadline. F6 named only the start and the retype, so I don't count that as a left twin.

Tool calls: none. My context already held every changed file and the summary.
