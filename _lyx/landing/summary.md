# Fix session-lifecycle, notice and halt bugs

Fixes the bugs collected on the `bugfix-sessions` board entry.
Each fix has a regression test that enters through the real CLI or engine wiring, with fakes only at the tmux, provider and LLM substrate.

## What changes

- **Typed prompts are submitted (#395).**
  `claudeengine`'s text-plus-submit sequences (`ComposeSend`, `ModelSwitchSequence`) type the text, settle, then send Enter as its own step.
  `sendVerified` then reads the input box through a new `InputBoxReader` provider capability.
  While the box still holds the sent text, it presses Enter again, at most twice, and then fails with an error naming the pending input.
- **An escalation halt spawns no reflection (#399).**
  A halt classified `escalation-to-human` writes `loom-halt.md` and reports `friction: skipped`.
  Other halt kinds reflect as before, and the terminal Friction-Reflect row still reads the note.
- **No superseded session runs beside its successor (#400).**
  Before a producer row starts a fresh shuttle run, shuttle removes the live strands of earlier runs of the same output-file set that the attach probe judged respawn-eligible.
  `recover-batch` removes its recovery strand on every terminal outcome, and `webster reset --to start` removes the live recovery strands of the batches it resets.
- **A clean step handoff is no longer a crash-resume (#396, #392).**
  A step that returns an error after persisting still records the handoff voucher.
  The `loom-crash-resume` note now carries the run-lock reading, the voucher reading and every history row.
- **The orch watcher reloads only after a fresh compaction (#390).**
  A compaction enters the reload sequence only when exactly one turn end follows it and the watcher is reading that turn end.
  A boundary with more turn ends after it moves `State.CompactionBaseline` past it with a log entry and types nothing.
  A reload after a compaction no longer retypes skills; only the reload after `/clear` does.
- **The orch session picks up newly deployed skills (#393).**
  Every reload sequence starts with `/reload-plugins`, persisted as its own step.
- **`lyx orch start` waits for a stopping watcher (#390).**
  A watcher that receives SIGINT or SIGTERM records `watcher_stopping` in the orch state.
  `orch start` waits for the lock within a bounded number of poll intervals while that record is present.
  Past the bound it reports the watcher live and says it is stopping.
- **A Burler review with invalid frontmatter is repaired in its own session (#386).**
  Every round's gate now parses the review file with `ParseReview` and re-prompts the reviewer with the parse error, inside the gate's existing attempt budget.
  The post-gate parse stays and fails the round only when the budget is spent.
- **Start verbs never attach a terminal.**
  `lyx loom start` and `lyx orch start` never attach or switch a tmux client, with or without `$TMUX`; attaching is only `lyx reed attach`.
  `--no-attach` stays as a hidden no-op because batten and existing callers pass it.
  Docs, stencils, help text and `doc.go` files that described the handoff are updated.

Closes #395
Closes #399
Closes #400
Closes #396
Closes #392
Closes #390
Closes #393
Closes #386

## Design decisions

- **Config key `submit_settle_ms`.**
  The settle between typed text and its Enter is read from `shuttleengine` config, default 300, so it can be tuned to the lowest value that works.
  It is an operator-approved exception to the board entry's no-new-config-keys rule.
  The confirmed submit retries a swallowed Enter, so a lower value is safe.
- **New verb `lyx burler validate-review <review-file>`.**
  It is the self-check verb `PATTERN-gate-self-check-parity` requires beside the new gate entry.
  It is read-only, calls `ParseReview`, and exits 1 with its error on failure.
  The gate entry, the verb and the post-gate parse call the same function.
  This is the only verb the task adds.
- **Extra Enters are bounded by the provider's reading.**
  An extra Enter is sent only when the input box holds the sent text.
  A draft of other text, a collapsed paste placeholder, an empty box or a running turn never gets one.
  A provider without the capability, or a reading that cannot tell, keeps the appearance-only check.
  A text long enough to collapse into a paste placeholder is therefore covered only by the pacing.
- **Strand removal is bounded by the attach probe.**
  The removal in shuttle only touches strands whose run declares the exact output-file set and that the probe judged respawn-eligible.
  A failed removal fails the start with an error naming the strand and `lyx reed remove`, and spawns nothing.
  `AttachIfLive` is a new entry for callers that only probe.
- **Webster removal only touches recovery strands.**
  Implementer and master strands are untouched.
- **Reload after a compaction skips the skills step.**
  This follows the board entry's premise that Claude Code re-injects invoked skills after a compaction.
  It supersedes the earlier statement in `internal/orchengine/doc.go`, which is updated.
- **`reedengine`'s client file and its test are deleted.**
  The attach code they held has no caller left.
  `StartAttachOnly` became `StartAlreadyRunning`, and `DecideStart`'s former attach-only branch changes nothing and reports the live strand and watcher.

## To check by hand

- **A1:** `/reload-plugins` exists in the deployed Claude Code and rebuilds the Skill tool's list without ending a turn.
  If it does not, the step is a no-op and a new skill stays unknown until a fresh `lyx orch start`.
- **A2:** Claude Code re-injects invoked skills after a compaction.
  If it does not, a compacted orch session misses a skill until its next `/clear` or restart.
- **Settle tuning:** run a few real `lyx shuttle send` deliveries against Claude Code at a lower `submit_settle_ms` than 300 before lowering the default.
- **Real terminal:** run `lyx loom start` and `lyx orch start` inside tmux and confirm neither moves the client.
- **Driver bootstrap timing:** every typed send now paces and confirms its submission, which adds about a second to the two sends `startLLMDriverArm` types.
  The test `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` in `internal/loomcli` has its fixed stub settle delay raised from 2s to 5s for that reason.
  The tmux and integration tests of `internal/loomcli` and the tmux run of `internal/webstercli` pass, but the full integration suite was not run.
