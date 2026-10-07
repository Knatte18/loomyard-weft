# Webster round 2 — fixer report

I fixed every finding in `round-2-review.md`, each in its own commit.
After the last fix, the plan's verify commands all pass: `go build ./...`, `go vet -tags llm ./...`, `go test ./...`, `go test -tags integration ./...`, and `go test -tags tmux ./internal/loomcli/... ./internal/webstercli/...`.

## Fixed

- **F1 (BLOCKING, design)**, `cc49d4c3c`: I reflowed the remaining comments to one sentence per line without changing their wording.
  The files are `internal/loomcli/smoke_starttail_test.go`, `internal/loomcli/smoke_test.go`, `internal/shedadapters/burler_test.go`, `internal/shuttleengine/attach_test.go` and `internal/shuttleengine/claudeengine/claudeengine.go`.
  The `claudeengine.go` header also now says the methods are pure functions of their arguments and the settle, which matches the `Claude` type's own doc comment.
  A final grep of the diff's added comment lines finds no mid-line sentence end and no line that ends mid-phrase.
- **F2 (BLOCKING, design)**, `e82538739`: the new untagged tests that touch no global state now call `t.Parallel`, and so do their subtests where each has its own fixture.
  The tests are `TestAttach_RemovesSupersededStrands`, `TestRunner_ReloadPlugins_PlaysSequenceOnALiveShuttleStrand`, `TestLoadConfig_SubmitSettleMS`, `TestInputBoxText`, `TestReloadPluginsSequence`, `TestCompactedSince_CountsTurnEndsAfterTheBoundary`, `TestStartEnvelopeFields_PinsSuccessEnvelope` and `TestRun_RecordsStoppingBeforeTheInFlightTickFinishes`.
  `TestRun_Send_ConfirmsSubmission` stays serial, with a comment naming the package-level `inputSleep` it replaces.
  `go test -race -count=3` passes over the three affected packages.
- **F3 (LOW, design)**, `071b37fce`: the reset verb now refuses only through `errors.As` on `RemoveRecoveryStrands`'s typed error.
  The unreachable fallback with no way forward is gone.
  `TestResetCmd`'s refusal step still pins the message.
- **F4 (LOW, consistency)**, `105f0771d`: two tests are renamed.
  `TestWatcher_AutoCompactionReloadsSkillsRoleAndPointer` is now `TestWatcher_AutoCompactionReloadsPluginsThenPointer`.
  `TestSmokeStart_AttachTailAddsNoOperatorStrand` is now `TestSmokeStart_NeverAttachesAndAddsNoOperatorStrand`.
  `docs/benchmarks/test-redundancy.md` still uses the old names; I left it alone because it is a measurement report of a past run.
- **F5 (LOW, scope)**, `0d67a8af6`: `start_watchdog_test.go`'s file comment no longer mentions a gate position under `--no-attach`.
  It now says that firing on every start is asserted by the smoke tier.
- **F6 (NIT, consistency)**, `a64e39808`: in `Watcher.Run`, the goroutine channels are now `runReturned` and `recorderExited`, and `recordStoppingOnCancel`'s parameter is renamed to match.
  The loop's `done` from `w.Tick()` no longer shadows a channel.

## Deferred

Nothing deferred.

## Notes

- Round 1's fixes still hold.
  `saveStateForStrand` keeps `WatcherStopping`, and the orch stencil headers describe the compaction sequence correctly.
- `gofmt -l` flags `internal/burlerengine/engine.go`, a misaligned `Round:` field in the shuttle spec literal.
  That misalignment already exists at the merge base `95902574`, so it is not part of this diff and I left it unchanged.
