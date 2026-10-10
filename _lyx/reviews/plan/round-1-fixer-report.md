# Plan-Review round 1 — fixer report

All seven findings were validated against the code and the spec and fixed.
Two new cards renumber the plan: the former cards 6, 7, 8 and 9 are now 7, 9, 10 and 11, and every card-number reference in the overview and the cards follows.
`lyx loom validate-plan` passes, with only the two informational `create-new-unit` notes it carried before.

## Fixed

- **F1** (BLOCKING): confirmed against `revive_test.go:13-86`, whose `reviveTestEngine` keeps `newTestEngine`'s nonexistent shell and calls `ensureServerAndSessionLocked` directly.
  Card 9 adds `internal/reedengine/revive_test.go` to `Edit:` and has `reviveTestEngine` set `cfg.Shell` to `sh` on POSIX.
  The sweep the finding asked for: among the untagged test files, only `revive_test.go`, `window_test.go` and `probe_test.go` answer `list-commands`; `TestOpenWindow` and the probe test call `probeCapabilityLocked` directly, never the boot's pre-tmux block, so `TestUp_BootValidation` and `TestEnsureServerAndSession_RevivalTrigger` are the only two affected, and card 9 now says so.
- **F2** (MEDIUM): confirmed; `probeCapabilityLocked` routes through `TmuxCmd`, which adds no `-f`.
  Card 9 now edits `TmuxCmd`, `TmuxCmd.serverArgs` and `Engine.probeCapabilityLocked`: an unexported `configFile` field leads the server flags with `-f <configFile>`, and the probe runs through a copy with `/dev/null` on POSIX, with the bound stated in `Intent`.
  `execHook` receives the argv without server flags, so the fake tmux tests stay as they are.
  The `tmux`-tier test's marker `.tmux.conf` also creates a session, which makes the marker assertion cover the probe as well as the spawn.
  The Shared Decision `probe-server-bound` is rewritten to match.
- **F3** (LOW): confirmed against `attach_test.go:85-87` and `:184-214`.
  The `ShellUnresolvable_PinsNothingAndStillChains` row is dropped; the shared `assertStrandOptionsReasserted` check also asserts that no shell pin was issued.
- **F4** (LOW): new card 8, `kit-config-text`, creates `tmuxkit#ConfigText` and edits `tmuxkit#serverConfig`, ahead of card 9, which now lists the export under `Uses:` and keeps the `tmuxkit.go` package-doc rewording.
- **F5** (LOW): card 5 no longer deletes `tickerPeriodFor`; new card 6, `drop-ticker-period`, deletes it and its `TestWatchState` step, with an `ImpactSummary`.
- **F6** (LOW): confirmed, and `TestWatchLoop_NonSentinelFailureDoesNotGoDormant` (`watchloop_test.go:913-930`) has the same shape, on failing re-applies.
  Card 5 now gives the real reason both tests pass, that the test ceiling keeps five backed-off ticks inside 300 ms, names that bound on `PollCeiling` in `watchdogTestTiming`, and rewords both tests' doc comments.
- **F7** (NIT): confirmed against `spawnwatchdog.go:1-9`; card 2 no longer names the file comment.

## Disputed

None.

## Deferred

None.
