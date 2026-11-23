```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check that the F1–F9 changes still agree with their surrounding docs, summary and sibling code.

**LOW — `internal/creelengine/doc.go:274`.** It says "A read message that expects a reply and has none is settled unanswered with the reason `holder-gone`". The fix adds a second sweep reason, `asker-gone` (`chase.go`, `sweep.go` `releaseGoneHolder`), and the package doc doesn't name it. The new `awaits-reply` verb and the removal sweep are documented here as well. A reader of the doc would not learn that a removed asker's questions are settled.

**LOW — stale doc comments in `internal/creelengine/sweep.go`.** The `sweepLoop` and `sweepNeeded` comments say the sweep runs "while a strand holds unread mail, a read message still awaiting a reply, or a queued inject". The `sweepSubjects` comment says it returns "the strands that hold unread mail, a chaseable message or have an inject queued". The fix also makes the sender of an open question a sweep subject (`isOpenQuestion(msg) && isStrandName(msg.From)`). All three comments still describe holders only.

**LOW — `_lym/landing/summary.md`, Tests section.** The fixer edited "What changes", "Design decisions", "Bounds" and "Merge danger", but left Tests untouched:
- It says `TestRun_Wait_ReplyHold` "checks four things", but the table now also has pane-busy and turn-start rows, and there are two new sibling tests for the failing-gate and pause paths.
- The `TestStore_AwaitsReply` row list omits the returned-question, foreign-mail and `asker-gone` rows.
- The new `creelcli` non-orch-pane row is missing.
- The new wiring assertions (`HasReplyProbe` in the loom, burler, webster and batten tests) are missing.
- The summary and the test list also name no `shuttleengine.Runner.HasReplyProbe`, which the fixer added to `run.go` as a production accessor.

**LOW — `internal/shuttleengine/doc.go`.** It still says a held turn end ends when no reply is awaited. The release now also requires an idle pane, which `summary.md` and `replyhold.go` state. The package doc is the one place that omits it.

**LOW — `chase.go` `noticeUnansweredLocked`.** The reason-to-text mapping covers only `holder-gone`. `asker-gone` would fall through to "was read and chased N times with no reply". It is unreachable today, because the sender is unlisted and the function returns early. But `settleUnansweredLocked` takes any reason, so the twin branch was not updated.

**Checked, no finding:**
- `cardFixerProgram = "GATE"` is the same shape as the other program constants (`BATTEN`, `LOOM`).
- The `burlercli` `runner` field and its `c.runner = runner` assignment agree with the new assertion.
- The wiring sites that were wired all got a `HasReplyProbe` assertion.

Further tool calls:
1. `grep` for `holder-gone` and `UnansweredHolderGone` across the repo: to find docs that list the unanswered reasons, which my context lacked.
2. `grep` in `internal/agentname/agentname.go`: to check program-name validation for `GATE`, since my context lacked it. The output showed only the function signatures, not the validation rule.
