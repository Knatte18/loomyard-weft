# Card 5 — never-ready-pane-tail

**Create:**
- `plan:internal/shuttleengine#lastLines` -> `func lastLines(capture string) string`

**Edit:**
- `internal/shuttleengine#paneTail`
- `internal/shuttleengine#Run.abandonStartup`
- `internal/shuttleengine#TestStartup_PaneNotLiveMidStartup`
- `internal/shuttleengine#TestStartup_UndismissableGateUntilWindowExpires`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`
- `internal/shuttleengine#TestStartup_SendFailuresTearDown`
- `internal/shuttleengine/doc.go`

**Intent:** A never-ready start's error ends with the last lines of its last startup capture (D4, as restated after Discussion).

`lastLines`, in `send.go`, holds `paneTail`'s cut: the last `paneTailLines` non-blank lines of a capture, right-trimmed, joined by newlines.
`paneTail` keeps its signature and its capture-failure sentence, and calls `lastLines` on a successful capture.

`abandonStartup` appends `. The pane's last lines:\n<lastLines(run.lastStartupCapture)>` to its returned error, the same suffix `withPaneTail` writes, exactly when all three hold: `cause` is nil, `run.state.Started` is false (the provider never reached `StartupReady`), and `run.lastStartupCapture` is non-empty.
That is `awaitStartup`'s pane-died, run-deadline and tick-cap exits.
The reed-status retry exits carry a cause and get no tail.
The skill-load death in `loadSkillsThenPrompt` has no cause, but its run is already `Started`, and its last startup capture is the ready screen from before the death, so it gets no tail.
Every cause-carrying teardown gets nothing added: a busy, in-draft or unlanded send cause already ends with the pane's last lines through `withPaneTail`, and the skill-loader lookup, prompt-offset persist and raw key or paste delivery causes report a mechanism failure with no screen.
`abandonStartup` keeps its one `fmt.Errorf` return, so `completionsignal_enforcement_test.go`'s pinned return count is unchanged.
Its doc comment states the tail rule in one sentence per line.

Tests:
`TestStartup_PaneNotLiveMidStartup` asserts, on every row, that the error ends with `"The pane's last lines:\n" + lastCapture` (`still booting...`).
`TestStartup_UndismissableGateUntilWindowExpires` asserts its error ends with `"The pane's last lines:\n"` and the gate capture's last lines, which pins the window-expiry exit.
`TestStartup_SkillLoadTurns`'s row "pane that dies during a load turn is a died startup" asserts its error does not contain `The pane's last lines`.
`TestStartup_SendFailuresTearDown` asserts on every row that `The pane's last lines` occurs at most once in the error, so no cause gets a second tail.

`internal/shuttleengine/doc.go`'s paragraph on the not-ready teardown ("A not-ready provider is torn down …") gains: a provider that never became ready reports its last startup capture's last lines at the end of the error.

**ImpactSummary:** `paneTail` keeps its signature and output; `abandonStartup`'s never-ready errors grow a multi-line suffix, reaching `StartGated`'s callers directly (burler and the seats engine copy that text into their own `StartError` fields) and the single-LLM producer through card 3's `Result.StartError`; none matches on the error's ending.
