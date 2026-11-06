```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set through the consistency lens (twins and siblings the changes touch).

Findings:

1. **LOW** — `internal/shuttleengine/run.go:788-800`, the `loadSkillsThenPrompt` doc comment.
   - Summary: it was not updated for the new open-turn deadline failure, so it reads as if a load turn can no longer fail the start by timing out.
   - Evidence: the comment says "one whose turn timed out are skipped … None of these fails or hangs the launch." Its only failure list is a dead pane and "every other failure … a skill-load send, a prompt-offset persist or a prompt delivery".
   - New behavior: `abandonLoad` now ends a start as `OutcomeTimeout` with a `*loadDeadlineError` when a turn is still open at the run's deadline.
   - The sibling comment on the same function was edited at line 795, the API-failure sentence, so this one was missed.

2. **LOW** — `internal/shuttleengine/doc.go:329-330` (not edited), the "Skill loading" paragraph.
   - Summary: "a skipped skill never fails or hangs it" now sits beside new text saying an open turn can fail the start at the run's deadline.
   - Evidence: the new lines just above say "a deadline passing first ends the start as timeout with the prompt never typed". Read alone, the unedited sentence "So a launch runs at most two load turns the API does not fail, and a skipped skill never fails or hangs it" can be read as contradicting that. It stays true only for a skipped skill, not for an open turn.

3. **LOW** — `internal/shuttleengine/engine.go:34` and the `Outcome` doc for `OutcomeTimeout`.
   - Summary: the `Outcome` docs name API-error from a skill-load turn but, from what my context shows, nothing says a start can now end `OutcomeTimeout` at a load turn.
   - Evidence: engine.go:34 was updated by the earlier commit for the model-rejected end. I did not read the `OutcomeTimeout` line itself, so this is unconfirmed.

4. **LOW** — `internal/shuttleengine/loadturn.go` `loadSkillTurn` doc, line 76, and `settleLoadTurn`.
   - Summary: the unedited first sentence still reads "until the first turn end …, the pane dies, or timeout passes".
   - Evidence: the edited lines below it say timeout ends only a turn no turn start was read for. The first sentence is now imprecise but does not contradict it.

Checked and consistent:
- `spec.go` `SkillLoadTimeout`, the doc.go "Skill loading" and "Awake deadline" text, the `Run.deadline`/`notifier`/`SetNotifier` comments and the PATTERN entries agree with the code.
- No remaining references to `loadTurnOpenBound` or "open bound" outside tests; the old row name is gone.
- `abandonLoad` still contains exactly one `classifyDeadlineExpiry` call. The completion-signal scan covers only `wait.go` and `attach.go`, so it is unaffected.
- `orchcli/start.go` passes `SkillLoadTimeout: HandoffTimeout`. I found no doc elsewhere that describes it as capping an open turn.

Tool calls (1 of 6):
1. `grep` for `HandoffTimeout|skill-load|skill load` across non-test Go, md and yaml outside `doc.go` and `loadturn.go`: to find twins and callers of the changed text that my context lacked.
