# bugfix-sessions: session-lifecycle, notice and halt fixes

All 18 batches reached `done`, each with a regression test through the real CLI or engine wiring.
Batch 01 was consumed from a report a crashed session left behind; the other 17 ran through begin-batch, an implementer fork and record-batch.
Head after the batches: `c93e0d20a`; head after the verify-gate fix: `1ea37b40f`.

## Typed input (batches 1-3)

- Batch 1 paces Claude text-plus-submit sequences: type the text, settle, then send Enter as its own step.
  The settle is configurable through `shuttleengine` config, and the CLI wirings that build the Claude engine were updated.
- Batch 2 adds the `InputBoxReader` capability to `shuttleengine` and its Claude implementation in `claudeengine`.
  Deviation: the interface is declared in `internal/shuttleengine/engine.go`.
- Batch 3 makes `sendVerified` press Enter again, at most twice, while the input box still holds the sent text, and fail with a pending-input error after that.
  Deviation: the new helpers sit in `internal/shuttleengine/run.go`.

## Halts, strands and handoff (batches 4-9)

- Batch 4: an escalation-to-human halt writes its note and spawns no reflection.
- Batch 5: the attach probe removes live respawn-eligible strands of the same outputs before a fresh run, through a new `AttachIfLive` for callers that only probe.
  Deviations: a test double in `internal/loomcli/parentreview_integration_test.go` and the audited-sites table in `internal/shuttleengine/completionsignal_enforcement_test.go`.
- Batch 6: `recover-batch` removes its recovery strand on every terminal outcome, including a refusal after the terminal check.
- Batch 7: `webster reset --to start` removes every live recovery strand first, through `RemoveRecoveryStrands`.
- Batch 8: a step that returns an error after persisting still records the handoff voucher, so a clean handoff no longer reads as a crash.
- Batch 9: the crash-resume note carries the lock and voucher readings and lists every history row.

## Orch watcher (batches 10-13, 15-16)

- Batch 10: the provider reports a `CompactionBoundary` with the turn ends after it and whether the read one is the newest.
- Batch 11: the watcher reloads only after a compaction with exactly one read turn end after it.
- Batch 12: adds the `/reload-plugins` provider sequence and `Runner.ReloadPlugins`.
  Deviations: `internal/shuttleengine/engine.go` and `session.go`.
- Batch 13: every reload types `/reload-plugins` first, and only a `/clear` reload retypes skills.
- Batch 15: `lyx orch start` never attaches or switches a tmux client; `--no-attach` stays as a hidden no-op.
  The `reedengine` client file and its test were deleted, and `StartAttachOnly` became `StartAlreadyRunning`.
- Batch 16: a signalled watcher records `watcher_stopping`, and `orch start` waits for it within a bound.
  Deviation: `internal/orchcli/cli.go`, for the injected `sleep` field.

## Loom start (batch 14)

`lyx loom start` never attaches or switches a tmux client and always prints the success envelope.
`--no-attach` is a hidden no-op, and the handover decision and its test were removed.

## Burler review gate (batches 17-18)

- Batch 17: every Burler round gates its review file on `ParseReview`, re-prompting the reviewer, through `CheckReviewFile` and a `ReviewGateEntry` appended to each round's gate spec.
  Deviations: `internal/shedadapters/attachprobe_test.go` and `internal/burlerengine/doc.go`.
- Batch 18: adds `lyx burler validate-review <review-file>`, the gate's self-check verb, with a table test that checks it agrees with the gate.

## Verify gate fixes

The gate's first attempt failed on `TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver` in `internal/loomcli`: `startLLMDriverArm` took about 2.5s, past the test's fixed 2s stub settle delay.
The fixer fork found the cause in cards 1 and 3, which make every typed send pace and confirm its submission and so add about a second to the two sends `startLLMDriverArm` types.
It raised `stubSettleDelay` in `internal/loomcli/integration_driverbootstrap_test.go` from 2s to 5s with a comment saying why, leaving the assertion unchanged.
Commit `1ea37b40f` (`fix:`).
The fork ran the tmux and integration tests for `internal/loomcli` and the tmux run for `internal/webstercli`, all passing; it did not run the full plan verify.

## Notes

Other deviations were files forks changed outside a card's declared file-ops, all informational.
