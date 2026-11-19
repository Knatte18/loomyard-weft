```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. For each function and branch the diff adds or changes, I ran one concrete case through the code.

**Per function: the case I ran, and whether it is correct**

- `driftengine.judgeAskSeed("judge-ab")` returns `"drift-judge judge-ab"`. Correct.
- `JudgeParentSeat.heldJudgeAsk`, with no `asks.json` and no lock: `state.ReadJSON` treats `ErrNotExist` on both the lock and the file as found=false with no error (`internal/state/state.go:112-133`). It returns false, nil. Correct.
  - This is the default production path, because `driftwiring.go:164` always wires a non-nil seat and the key is off by default.
- `heldJudgeAsk` with the ask record unreadable returns the error. `parentAgreement` passes it on, so the judge's table build fails and so does `agent.Call`. Correct: the error is surfaced, not swallowed.
- `agreementGate`:
  - With judgement.yaml missing, it returns a gate error. The judgement entry runs first and fails earlier. Correct.
  - With no ask held:
    - `Resolve` errors: the ask is recorded no-parent and the entry passes. Correct.
    - `Resolve` returns not-recorded: same as above. Correct.
    - `Resolve` works: the ask is recorded pending-post with deadline now+wait, posted and marked open, and the entry fails with the await text. Correct.
  - With a held ask still pending-post, it posts again and then judges. Correct.
  - With a held ask in no-parent, timeout or not-sent, `agreement` falls out of the switch and passes. Correct.
- `sendJudgeAsk` when `RecordParentAsk` fails returns the error, which becomes a gate error. Correct.
- `postJudgeAsk`:
  - When `Post` fails, the ask is ended no-parent and returned no-parent, and the entry passes. Correct.
  - When `Post` fails and `EndParentAsk` fails too, the record stays pending-post while the entry passes. The next evaluation (if any) posts again. The failure is logged, and it mirrors `shedadapters.postParentAsk`. Acceptable.
  - When `Post` works and `MarkParentAskPosted` fails, the record stays pending-post. The next turn-end evaluation re-posts the same line, so the parent may get duplicate mail. Logged, and it mirrors the review seat. Acceptable.
- `agreement`:
  - Open, with now 1 minute before the deadline: fails with the await text. Correct.
  - Open, with now equal to the deadline: `EndParentAsk` sets it to timeout and the entry passes. Correct.
  - Open past the deadline, when an answer landed first: `EndParentAsk` returns the done ask and the function recurses into the done branch. Correct.
  - When `EndParentAsk` returns found=false: `ended.State` is `""`, the switch falls out and the entry passes. Acceptable.
  - Done with a satisfied report: passes. Correct.
  - Done with findings and judgement.yaml written before `AnsweredAt`: fails, asking for a rewrite. Correct.
  - Done with findings and the file rewritten after the answer: passes. Correct.
  - Done when `ReadParentReport` fails: gate error. The report is lym's own file. Acceptable.
  - Edge case: the rewrite check compares `ModTime` with `AnsweredAt`. On a filesystem with coarse mtime resolution, a rewrite made within the same tick as the answer reads as "not after", so the judge is held and spends an attempt. See F3.
- `awaitParentFindings` with a pending-post ask: the deadline is set, because `sendJudgeAsk` sets it before pending-post. Correct.
- `JudgeAskLine` with ask 3: the tag, slug, file, summary, deadline and both answer forms are all present. Correct.
- `judgementSummary`:
  - With an empty judgement it returns `"no rulings"`. Correct.
  - Cells only take `matches` or `does not match` (`judgement.go:82,188`), so counting every non-no-match cell as a match is exact. Correct.
  - Files only take `none` or `loosened` (`judgement.go:56,231`), so counting every non-loosened file as `none` is exact. Correct.
  - A file holding two uncaused checks is named once, thanks to the `break`. Correct.
  - Cases are named with their ruling. Correct.
- `DriftVerify.parentAgreement`:
  - With a nil seat, nothing is carried. Correct.
  - With the key on, the entry is carried with wait = `JudgeParentTimeoutMin` minutes. Correct.
  - With the key off and no ask held, nothing is carried. Correct.
  - With the key off and an ask held, the entry is carried. Correct.
  - When `Config()` errors, the error is returned and the table build fails. Correct.
- `DriftVerify.tableWithParent` appends the parent entry after the judgement entry. Correct.
- `runAgent`: the table closure captures the request. Correct.
  - `settleOpenSession` still probes with `tableFor`, which has no parent entry. Probing looks for the chair, so the missing gate entry does not matter. Correct.
- `driftengine.LoadConfig` with `judge_parent_timeout_min: 0` refuses, naming the key and `lym config drift`. With 5 it loads. Correct.
- `loomengine.LoadConfig` with `code_review_parent_timeout_min: 0` refuses. Correct.
- `loomengine.parentReviewTimeout` with the discussion perch and with the code perch returns each one's own key. With any other perch it panics, and the only caller, `baseParentSeat`, passes just those two. Correct.
- `CapParentReviewWait`:
  - Code perch, room 227m, want 45m: returns 45m. Correct.
  - Code perch with a 13-minute chair: room ≤0, and the reason names `code_review_parent_timeout_min`. Correct.
- `shedadapters.parentAskDue`:
  - On an AskEachRound seat it returns true before ever reading `RecordWrittenAt`, which is nil on the code seat. Correct.
  - Discussion seat, where the latest ask is a code-review or drift-judge ask with another seed: it returns true, and the ask goes out although the record was not rewritten. See F2.
- `askSubject`:
  - Round 1, or a seat that does not ask each round: returns the bare subject. Correct.
  - Round 2: names round 1's review and fixer-report paths. Round 2 exists only after round 1's fixer and Bouncer have run. Correct.
- `ParentAskLine` with the discussion tag reproduces the old line byte for byte, so `TestParentAskLine`'s literal still matches. Correct.
- `postParentAsk` builds its line through `askSubject(ask.Round)`, so a pending-post ask that is posted again carries the same subject. Correct.
- `loomcli.parentSeats`, `discussionParentSeat`, `codeParentSeat` and `baseParentSeat`: the key is captured per seat and `awaitCap(perch)` is resolved once per seat. Correct.
- `askParentSetting`:
  - When the lookup errors, it logs the key it was handed and returns false. Correct.
  - When the value is a non-boolean `"yes"`, it returns false. Correct.
- `loomSegmentSettings`: a perch absent from the map gets `Parent: nil`, because a map lookup of a missing key yields nil. Correct.
- `driftDeps`' `JudgeParent`: `Files` comes from the anchor, the same record the review seats use, and the post carries the `loomProgram` signature and the run tag. Correct.

**Findings**

- **F1 — LOW, design.** Location: `internal/driftengine/judgeparent.go:146-147` (`postJudgeAsk`, the `MarkParentAskPosted` failure branch).
  - Summary: when the mark fails, the returned ask says open but the record keeps it pending-post. Every later turn-end evaluation then takes the `case ask.State == burlerengine.ParentAskPendingPost` branch at `judgeparent.go:270-271` and posts the `[drift-judge request]` line again. The parent gets one more copy each time the judge ends a turn, until a mark succeeds.
  - Evidence: `judgeparent.go:144-149` and `270-271`; `burlerengine/parentreview.go:194-208`. The review seat behaves the same way, so this is a known trade-off, but the judge's entry is evaluated once per turn end, not once per round.
- **F2 — LOW, design.** Location: `internal/shedadapters/parentseat.go:120` (`parentAskDue`).
  - Summary: the code-review and drift-judge asks now share the run's one ask record (`ParentReviewFilesOf(location.AnchorPath())`, at `parentseatwiring.go` `baseParentSeat` and `driftwiring.go:164-167`). A Discussion-Review generation re-entered by a goto keeps its seed, yet finds a code-review or drift-judge ask as `latest`. `latest.Seed != seed` then returns true, and the discussion seat asks the parent again although the decision record was not rewritten.
  - Evidence: `parentseat.go:118-121`, and the `roundSeed` doc in `parentseat.go`: "a goto, a resume and a restarted row keep it".
- **F3 — LOW, design.** Location: `internal/driftengine/judgeparent.go:342-346` (`agreement`, the findings branch).
  - Summary: the rewrite check `!info.ModTime().After(ask.AnsweredAt)` compares the judgement file's mtime, at the filesystem's resolution, with the answer verb's `time.Now()` at nanosecond precision. A rewrite inside the same coarse mtime tick as the answer reads as not rewritten. The judge is then held again and spends one of the `parentAgreementAttempts` (3). Exhausting them is a shed Stuck.
  - Evidence: `judgeparent.go:346`, `burlerengine/parentreview.go:338` (`ask.AnsweredAt = now`) and `judgeparent.go:211` (`parentAgreementAttempts = 3`).

I found no wrong result, swallowed error or inconsistent state on the main paths. The timeout, no-parent and failed-post paths all leave the record and the gate result in agreement, except F1's mark-failure window.

Tool calls:
1. `internal/state/state.go` `ReadJSON` and the ruling constants in `driftengine/judgement.go`. My context did not show how a missing record or lock is handled on the default key-off path, nor the closed sets `judgementSummary` counts against.
