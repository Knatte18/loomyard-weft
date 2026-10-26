```yaml
lens: code-pattern-fit
outcome: reported
```

Lens: code-pattern-fit. Checking the diff against the repository's rules (PATTERN entries and CLAUDE.md). I found no pattern violations.

**Rules checked**
- `PATTERN-shuttle-provider-seam`: **held.**
  - `wakesOnBackground` (`hold.go`) goes through `meetsAll`, which asserts `RequirementSupporter`, not a concrete engine.
  - The diff adds no `allOutputFilesExist` call site and no negative-verdict return site in `wait.go`, `attach.go` or `stop.go`.
  - The two completion-signal scan pins (`completionsignal_enforcement_test.go`) therefore need no update.
  - The start retry sits above `abandonStartup`'s verdicts and does not finalize a verdict itself.
- `PATTERN-config-layers`: **held.**
  - `start_retries` is declared in `template.yaml`.
  - It is classified `CategoryNone` in `ConfigSpec` (`config.go`).
  - `LoadConfig` refuses a negative value.
  - The `Config` field has a doc comment.
- `PATTERN-spawn-observability`: **held.**
  - Each EAGAIN retry is logged at Warn with the argv (`overlay.go`, `retryExecAgain`).
  - Each start retry (`run.go`) and each removal retry (`wait.go`) is logged at Warn.
  - The skipped held notice is logged at Info.
  - Retries are bounded, so the Level policy for loops is not an issue.
- `PATTERN-test-speed` and `PATTERN-test-isolation`: **held.**
  - Reed's retry test injects `TmuxCmd.pause`, a field, not a package variable.
  - The shuttle start-retry and removal-retry tests use `newFakeClock`.
  - The new tests call `t.Parallel()`.
- `PATTERN-shell-guard-table`: **held.** No pane-shell string is built in the diff.
- `PATTERN-daemon-wakeups`: **held.** The retry waits are bounded (about 1.75 s for reed, 5 s for the start retry, 3 s for removal) and are not unbounded polls.
- CLAUDE.md "Docs land in the same commit": **held.**
  - `shuttleengine/doc.go` and `reedengine/doc.go` describe the retry, the removal retry, the head/tail capture and the held-notice skip.
  - The `template.yaml` comment covers the new key.
  - The `overlay.go` and `hold.go` file headers are updated.
- Declared PATTERN changes: **held.** The directive declares none, and the range touches neither `PATTERN.md` nor `pattern/`. The EAGAIN retry at reed's exec seam and the held-notice skip are not new cross-cutting invariants.
- Commit messages and departures: **held.** There is no `Pattern-Departure:` trailer, and none is needed because no entry is departed from.
- CLAUDE.md "Notes go in git" and "Agents run in interactive tmux": **held.** Nothing in the diff conflicts with either.
- Comment conventions: **held.** The new comments follow the repository's style, with no end-of-line comments, and the diff adds no block comments. Line wrapping is left to `lint-comments`.

**Findings:** none.

**Tool calls**
1. `grep` for `allOutputFilesExist` and the completion-signal scan, plus `git diff --stat` on `PATTERN.md` and `pattern/`: my context lacked the scan test's pinned sites and whether the range touches PATTERN.
2. `git diff` on the test files filtered for sleep, clock and `Parallel`: my context lacked the test bodies.
