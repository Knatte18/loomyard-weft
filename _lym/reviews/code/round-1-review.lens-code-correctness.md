```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I traced each function and branch the diff adds or changes against one concrete input, and checked whether the result is correct.

**Per function or branch**
- `reedengine.NewOver` with a zero `Outside`: `paneAge`, `capMemory` and `newGUID` keep their real defaults and `toldTmux` stays false, so `New` behaves as at the base. Correct. With only `Tmux` set: `execHook` and `floorLine` are set, and `TmuxVersion=""` admits any version. Correct. Empty `EnvScrub`: it panics, but the message still says "New needs", which is misleading when `NewOver` is the caller (NIT).
- Told-tmux branch in `ensureServerAndSessionLocked` (`lifecycle.go:472-479`): `slices.Index(argv,"new-session")` is never -1, because `serverSpawnArgv` always adds it. The call goes through `tmux.run`, which checks the floor and passes `stdin=nil`. On a hook error it returns "start tmux: …". Correct. It drops `-v` debug flags, `cmd.Dir` and the scrubbed env; none of these matter to a hook.
- `capPaneMemoryLocked` with `e.capMemory` failing: it logs a Warn and returns, as at the base. Correct.
- `addStrandLocked` with `e.newGUID` failing: it returns "generate guid: …". Correct.
- `SessionLiveness` and `SessionRegistrar`: absent strand, `PanePID` 0, status error and register error all behave as at the base. Correct.
- `Recorder.tmux` with `capture=false`: `out` is cleared, and an exit-1 error records `Exit=1`. It never records `stdin`, which is part of finding 2.
- `formatPIDs` on `"$0|123|1795807440"` with format `"#{session_id}|#{pid}|#{start_time}"`: positions line up and it yields `[123]`. Correct. With no format arg it yields none. Correct.
- `pidPlaceholder` / `placeholdCall` with a kept pid of 1000 while `tmux-1000` or `user-1000.slice` appear in output: those uid tokens would also become `{pid:N}`. Wrong in an edge case (finding 5).
- `Bindings.placehold` with a short user or host name such as `tr` (equal to `Shortname`): every substring `tr` becomes `{user}`, including the strand names `tr:pr-reviewer`. Wrong in an edge case (finding 5).
- `Bindings.bind` and `replayPID` on `{pid:3}`: gives 2^30+3, above the kernel pid range. Correct.
- `Recorder.Recording` with `-V` or `display-message` failing: it returns a wrapped error. Correct.
- `Replay.answer`:
  - Mutating call after reads: the forward scan skips reads and matches. Correct.
  - Mutating call that is reordered or skipped: it hits a non-read, breaks, and is reported unanswered. Correct.
  - Read after a mutation, where the identical read was recorded only before that mutation: the backward scan returns the pre-mutation answer. Wrong (finding 1).
  - Recorded mutations the code never makes at the end: nothing reports them (finding 3).
- `Call.read`: it classifies by `Args[0]` alone. The one chained call (`tmuxconf.go:126`) is read-only, so correct today. A chain that starts with a read verb and holds a mutation would be misread (LOW, folded into finding 1).
- `Replay.tmux`: ignores `stdin` (finding 2).
- `Replay.paneAge` with no answer: returns `0,false`, which reed reads as "inside grace". The call goes into `unanswered`, so `PlaySession`'s cleanup fails the test. Correct.
- `answerErr` / `exitError`: reed's exit-1 check uses the `ExitCode()` interface (`overlay.go:341`), and the recorded text matches `wrapTmuxError`'s `"%w: %s"`. Correct. `stopserver.go:23` and `buttonclick.go:298` use `*exec.ExitError` and would see a replayed exit as a plain error; neither path is in the current recordings. Correct for the scenarios.
- `PlaySession` with an unknown name: `t.Fatalf` runs before `replay` is used. Correct.
- `countedSessions.StopStrand`: `beforeStop` and `afterStop` each run once, and `afterStop` runs even when the stop fails. Correct.
- `blockStarts`, `blockRegistration`, `blockStatus`: each produces a real permission or ENOTDIR failure in the module that reads that state. Correct when not run as root.
- `prreview_wait_test` seeding after `tt.setup`: no setup blocks reed before the seed. Correct today.
- `"session-ends"` scenario where no poll reads dead within `paneEndPolls`: the loop ends with no "ended" reading and no failure (finding 4).

**Findings**

1. **MEDIUM**, design, `internal/testkit/tmuxrec/tmuxrec.go:1006-1011`. The read fallback in `Replay.answer` scans back over the whole transcript, crossing recorded mutations. Suppose a read was recorded only before mutation M and is then made after M. The replay answers it with the pre-M server state, although the doc comment says "nothing changed since that answer". A row that reads state after a change therefore gets a stale answer instead of failing. The backward scan should stop at the latest non-read call before `r.next`. `Call.read` judging a chained argv by `Args[0]` alone is the same misclassification risk. Evidence: `tmuxrec.go:1006-1011`, `tmuxrec.go:774-783`.
2. **MEDIUM**, design, `internal/testkit/tmuxrec/tmuxrec.go:941` and `record.go:55-63`. The replay drops `stdin` and the recorder never stores it, so `matches` cannot compare it. A changed text in reed's paste (`load-buffer -b … -`, `io.go:135,147-151`), which appears in every recording, would still replay as a match. That is a mutating call whose content can change without detection.
3. **LOW**, design, `internal/testkit/tmuxrec/session.go:635-639`. `PlaySession`'s cleanup checks only calls the recording could not answer. If the code stops making trailing recorded mutations, such as a final `kill-pane`, the replay passes, and `TestRecordings_Replay` never asserts that `next == len(calls)`. Only the readings comparison guards it.
4. **LOW**, design, `internal/testkit/tmuxrec/recordings_test.go:363-370`. If no poll reads the pane dead within `paneEndPolls`, the loop exits silently. With `-tmuxrec.write`, the rewritten recording then lacks its "ended" step, and nothing fails. The loop should `t.Fatalf` when it runs out of polls.
5. **LOW**, design, `internal/testkit/tmuxrec/tmuxrec.go:851-862` and `record.go:187-205`. Placeholding replaces raw substrings and digit tokens:
   - A user or host name that occurs inside other text, such as `tr` inside `tr:pr-reviewer`, rewrites that text into `{user}` or `{host}`.
   - A kept pid that equals another number in the output, such as the uid 1000 in `tmux-1000` or `user-1000.slice` (`session-start-stop.json:332,838`), turns that number into a pid placeholder.
   Either corrupts a recording.
6. **NIT**, consistency, `internal/reedengine/lock.go:113`. `NewOver` panics with "reedengine: New needs a non-empty EnvScrub", naming `New` even when `NewOver` is the caller.

**Tool calls**
- Reed's exit-code and error-text handling and `retryExecAgain`: needed to judge whether `exitError` and the recorded error text match what reed reads.
- The chained and failing calls in the recordings, read with jq: needed to judge whether `Call.read`'s `Args[0]` classification misfires on the current data.
- The chained `show-options` call and its source in `tmuxconf.go`: needed to confirm that chain is read-only.
