```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, meaning what the diff forces outside the lines it touches.

**Per-change set**

- **Escape sent by `dismissPicker` (`send.go:343-368`, `claudeengine/session.go:164-196`).**
  - Set found: `PATTERN-pane-text-injection` (`PATTERN.md:66`, `pattern/PATTERN-pane-text-injection.md`) and `pattern/PATTERN-shuttle-provider-seam.md`.
  - Not covered. The pane-text-injection entry says an interrupt key reaches an agent pane only through shuttle's two pane guards. It names the guarded picker close, which is only for the rewind picker, as the one exception. Its "Explicit key deliveries stay outside the rule" list does not include the dismissal. The diff sends Escape, which is the provider's interrupt key (`InterruptSequence` now returns `escapeKey` too), through a third path. That path has its own guards (`RunningTurn`, operator activity) and none of the ready-agent-pane check or the post-key re-capture of the guarded close. No `PATTERN.md` or `pattern/` file is in the diff, and there is no `Pattern-Departure:` trailer on the commit. The entry also says text lym types never "closes a menu the operator has open"; the dismissal closes a picker, guarded only by the quiet window.
- **`SessionBusyError.NoInputBox` (`send.go:25-30`).**
  - Set found: the constructors at `send.go:209` and `conditionedsend.go:71`, the consumer at `wait.go:555`, and the doc at `doc.go:345`.
  - The `awaitIdleSession` and `conditionedReading` constructors are covered. The other `SessionBusyError` constructors, such as the active-operator hold at `send.go:179` and the early `requireReadyAgentPane` returns, leave the field unset. That is correct for them.
  - `IdleProbe.NoInputBox` is a separate, already existing field (`session.go:279`, `engine.go:453`), so there is no conflict. Covered.
- **`DismissiblePickerReader` capability (`engine.go:394`).**
  - Set found: `claudeengine/session.go:29` (assertion), `fakes_test.go` (`dismissPickerEngine`), `doc.go:342` and `claudeengine/doc.go:236`. Covered.
  - The provider-seam entry holds: the Claude wording is only under `claudeengine`.
- **`busyReading`, `clearStrayCharacter` and `dismissPicker` signatures (new `picker` parameter).**
  - Set found: the grep shows every caller is in `send.go` and all are updated. Covered.
- **`noBoxReprompts` field (`run.go:449`).**
  - Set found: resets at `wait.go:347`, `wait.go:558`, `wait.go:582` and `send.go:919`. Every site that clears `unsentReprompt` also clears it, so the field comment holds. Covered.
- **`maxNoInputBoxReprompts` and the new finalize path.**
  - Set found: the `wait.go` file comment (line 33), the `handleGatedBoundary` doc comment, and `doc.go:414`.
  - Covered. There is a gap in `PATTERN-shuttle-provider-seam` and the wait.go completion-signal tripwire (`completionsignal_enforcement_test.go`): the new `finalize(OutcomeDone)` call site in `wait.go` is a positive outcome, so it needs no `allOutputFilesExist` check. I could not confirm that the tripwire's audited site list accepts the extra `finalize` call, because I did not read the test.
- **`escapeKey` constant (`startup.go:223`).**
  - Set found: `InterruptSequence`. Other literal `"Escape"` sites, such as the trust-gate dismissal in `startup.go`, were not checked for the repeated literal, so I did not confirm whether `PATTERN-named-constants` is satisfied there.
- **Fixture `testdata/pane-autocompact-picker.txt`.**
  - Set found: the `TestDismissiblePicker` row. Covered.

**Findings**

1. **BLOCKING, class `design`, `internal/shuttleengine/send.go:343-368`.** `dismissPicker` adds an Escape path, the provider's interrupt key, that `PATTERN-pane-text-injection` (Guarded interrupt section, `pattern/PATTERN-pane-text-injection.md:32-38`) does not list as an exception. The diff does not edit `PATTERN.md` or `pattern/PATTERN-pane-text-injection.md` and carries no `Pattern-Departure:` trailer. Land a PATTERN change in the same diff that names the dismissal as a bounded exception with its guards, or justify it with a trailer.
2. **LOW, class `consistency`, `internal/shuttleengine/claudeengine/startup.go:223`.** The trust-gate dismissal's Escape literal, if any, was left unchecked against the new `escapeKey` constant. I did not search for other `"Escape"` literals in `claudeengine`. Treat this as unverified.

**Tool calls**
1. `grep` of the shuttleengine and cmd tests for interrupt-key scans, plus `PATTERN.md` for the pane-text-injection line. My context lacked any scan test that enforces interrupt keys, and the grep found none. It also gave the entry's exact wording at `PATTERN.md:66`.
