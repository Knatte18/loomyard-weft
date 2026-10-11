# Card 3 — startup-teardown-pane-tail

**Create:**
- `plan:internal/shuttleengine#lastLines` -> `func lastLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#flakyStatusReed`
- `internal/shuttleengine#skillReed`
- `internal/shuttleengine#skillReed.CapturePane`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_CaptureAlwaysErroringUntilWindowExpires`
- `internal/shuttleengine#TestStartup_MechanismFailure`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`

**Intent:** A never-ready start's error ends with the last lines of its last startup capture (D4, as the parent restated it).
`lastLines(capture)` is `paneTail`'s cut extracted: the last `paneTailLines` non-blank lines of `capture`, right-trimmed and joined by newlines; `paneTail` captures the pane and returns `lastLines` of it, or its capture-failed sentence, so every send failure reads as before.
`abandonStartup` appends `. The pane's last lines:\n<lastLines(run.lastStartupCapture)>` as the final element of the returned error when, and only when, `cause` is nil, the provider never reached `StartupReady` (`run.state.Started` is false, read under `recordMu`), and `lastLines` is non-empty.
`Started` is the discriminator because `checkLivenessTick` persists it at the ready probe.
So the teardown of a pane that dies during the skill-load turn, which has no cause but a ready screen as its last capture, gets no tail.
A teardown with a cause adds nothing: a busy, in-draft or unlanded send cause already ends with `withPaneTail`'s lines, and the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error report a mechanism failure with no screen.
The error still wraps `ErrNotStarted` and the cause.
The tail is built into the existing format string and arguments ahead of the one `return result, fmt.Errorf(format, args...)` statement, which stays the function's only return carrying `Errorf`, so `completionsignal_enforcement_test.go`'s audited sets stay as they are.
What it lets through: nothing new; it adds diagnostic text only, and a start with no recorded capture (every capture failed) gets no tail.
`abandonStartup`'s doc comment names the tail and its trigger.
`flakyStatusReed` gains a `failFrom int` field: when positive, every `Status` call after the first `failFrom` fails with `err`, so a run can record a capture on its first tick and then exhaust the status-retry cap.
`TestStartup_PaneNotLiveMidStartup` asserts for every row that the error ends with `The pane's last lines:\n` plus the capture it records (`still booting...`); the cut itself is the one `paneTail` already pins through the tall-frame send-failure test in `run_test.go`, since `paneTail` now calls `lastLines`.
`TestStartup_CaptureAlwaysErroringUntilWindowExpires` asserts the error holds no `The pane's last lines:` text, since no capture was recorded.
`TestStartup_MechanismFailure` gains a row where the reed status fails from its second call with a capture already recorded (`failFrom: 1`, a pending capture) and asserts the error wraps the status error and holds no `The pane's last lines:` text, so a cause-bearing never-ready exit gets no tail.
`skillReed` gains a `ReadyCapture string` field and its `CapturePane` returns it ahead of the typed texts it returns today, so a test using `skillReed` can give the startup probe a non-empty ready screen; the default empty value leaves every existing test as it was.
`TestStartup_SkillLoadTurns` gives its pane-dies-during-a-load-turn row a ready capture through the fixture's configure hook (a new row field `capture` setting `ReadyCapture`) and asserts the error holds no `The pane's last lines:` text; with the non-empty ready capture recorded and `Started` true, a tail added there would show.
`TestStartup_SendFailuresTearDown` sets `ReadyCapture` on its `skillReed` and asserts, for every row whose error is expected, that `The pane's last lines:` occurs at most once, so no cause gets a second tail.

**ImpactSummary:** `abandonStartup` has the `awaitStartup` never-ready exits and `loadSkillsThenPrompt` as callers and keeps its signature; `paneTail`'s behavior is unchanged and its send-failure callers keep the same text.

**Commit:** `3: startup-teardown-pane-tail`
