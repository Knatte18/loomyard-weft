```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Behaviour → pinning test**

- **Replay matches a tmux call on its stdin.** Pinned by the `OtherText` row of `TestReplay_MatchesAPasteByItsTextWhateverBufferItNames` (`internal/testkit/tmuxrec/recordings_test.go`), and by `TestRecordings_Replay` over `session-paste`, which checks unanswered calls, readings and remaining calls (`recordings_test.go:16-38`).
- **A replayed paste matches whatever buffer name it draws.** Pinned by the `AnotherFreshBufferName` row, and by the `session-paste` replay, where real reed draws a random name.
- **A call naming two buffers is refused.** Pinned by the `TwoBufferNamesInOneCall` row.
- **A call naming no buffer matches as before.** Pinned by `TestRecordings_Replay` over every older recording.
- **Recording side, buffer names placeheld before pids, stdin keeps its numbers, bound paths placeheld in stdin.** Pinned by `TestPlaceholdCall_HoldsAPastesBufferNameAndKeepsTheNumbersOfItsText` (`record_test.go`).
- **The Recorder keeps stdin.** Pinned only at tier2 (`TestRecordings_RealTmux` writes it). At tier0 it shows only through the stored recording's `stdin` field. Acceptable for a recorder.
- **`bindCall` binds placeholders in stdin.** None. No recording's stdin holds a placeholder, and the new replay test passes `Bindings{}`.
- **`describe` adds the stdin text to a failure message.** Exempt: the text is only read by a human.
- **`tmuxrec.SessionSpec` is exported, and the scenarios start under it.** Pinned by `TestRecordings_Replay`: a different spec would leave calls unanswered.
- **`reedengine.InjectBufferPrefix` feeds both reed and the kit's pattern.** Pinned by the `session-paste` replay: a drawn name the pattern missed would go unanswered.
- **`IdleSession` builds a replayed session that never skips.** Only checked for being non-nil, by `TestFullEnv_FillsEverySeam` (`envkit_test.go:60`). No test drives it.
- **envkit's PR-Review and sampler liveness and registration seams read reed's strand table.** None. The tier1 loomcli test builds its own seams over `PlaySession` instead.
- **`newSession` falls back to the shell's bare name when the shell is missing.** None.
- **landingshed's shared `MailAgent` now fails the test when called.** It still pins "mails nothing" for the closed-pull-request, ended-review and runner-crash rows: a mail in any of them fires `t.Errorf` (`prreview_test.go:717-720`). Same behaviour, no looser.
- **A refused head is mailed once per head over three wakes.** Pinned by `ARefusalIsMailedToTheSessionOncePerHead`: gates == 1, exact mail list to that strand, `MailedHead == "head"`, wait cancelled.
- **A cleared mailed head is gated and mailed again.** Pinned by `AClearedMailedHeadIsGatedAndMailedAgain`: gates == 2, two exact mails.
- **A mail creel refuses halts the row and records no mailed head.** Pinned by `AMailCreelRefusesHaltsAndRecordsNoMailedHead`: exact reason, Stuck with halt, `MailedHead == ""`.
- **A refused, mailed head is not pushed, and its `GatedHead` stays put.** None since the move (finding 1).

**Findings**

1. **MEDIUM**, consistency (loosened check), `internal/loomcli/prreview_mail_integration_test.go:278-318`
   - Moving the three mail rows out of the landingshed table dropped assertions the table's shared loop held for them, and nothing holds them now:
     - The push count must be zero: base `prreview_wait_test.go:952-958` with `noEntryPush`, and the first row's name ended `...AndPushesNothing`.
     - The record is compared whole, so `GatedHead` stays `h1` (base lines 959-966).
     - The wait mark is set and then cleared (base 981-983).
     - Every sleep falls between a second and a minute (base 972-976).
     - The parent lock store stays empty (base 984-987).
   - The new test checks only gate count, mails, `MailedHead` and the outcome. envkit's `PushTask` is `func() error { return nil }` (`envkit.go:262`), and nothing counts its calls.
   - So a regression that pushed the refused head, or advanced `GatedHead`, after mailing would pass.
   - No `Loosened-Check:` trailer is on any commit in the range (checked against the `git log` output).
2. **LOW**, design (coverage), `internal/testkit/tmuxrec/tmuxrec.go` (`bindCall`, `bound.Stdin = bindings.bind(call.Stdin)`)
   - Binding placeholders in a recorded stdin has no test.
   - `session-paste.json:1247` holds no placeholder in its stdin, and `TestReplay_MatchesAPaste...` replays with `tmuxrec.Bindings{}`.
   - Dropping the line passes every test, yet a paste of a path under `{hub}` would then never match.
3. **LOW**, design (coverage), `internal/testkit/envkit/envkit.go:490-499` (`replayedSessions`) and `internal/testkit/tmuxrec/session.go` (`IdleSession`)
   - No test drives the new envkit seams. Only their non-nil fill is checked (`envkit_test.go:60`).
   - The added behaviour, liveness and registration over the replayed reed's strand table, has no row that would fail if it were wired wrong. For example, `SessionLive` built over another reed, or `register` writing outside `dir`.
4. **LOW**, design (coverage), `internal/testkit/tmuxrec/session.go:118-123`
   - The shell fallback in `newSession` has no test. It changes `PlaySession` and `RecordSession` too.
   - Nothing pins that a host without the shell builds a session, or that reed's first start there fails.

Tool calls:
1. Grepped `recordings_test.go` (`TestRecordings_Replay`'s checks) and `envkit_test.go` (which tests touch the session seams). My context had the diff hunks of these files, not the replay loop's assertions or envkit's test list.
