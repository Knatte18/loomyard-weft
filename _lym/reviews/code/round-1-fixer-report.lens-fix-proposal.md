```yaml
lens: fix-proposal
outcome: reported
```

Task: challenging the review's proposed changes that were carried out as proposed.

**Findings**

1. **BLOCKING (F2).** Changed text: `withSessionEnd` and `endSessionLine` in `internal/shuttleengine/open.go`, applied in `launch` (`run.go`) and `relaunchUnderAccount` (`accountrelaunch.go`).
   - **Summary:** an agent whose provider exits during startup now removes its own strand, which breaks the startup teardown and the start retry.
   - **Case:** the provider exits before it is ready (bad model id, auth failure, crash). The pane line runs `cmd; touch; lym reed remove --detach; exit`, so the strand leaves reed's table before shuttle tears the run down.
   - `abandonStartup` then calls `removeStartupStrand`, which calls `reed.RemoveStrand` unconditionally (`wait.go:914-922`).
   - Reed answers `ErrUnknownStrand` for a guid it no longer holds (`internal/reedengine/strand.go:470-473`).
   - **Result:** it retries 3 times with 1s and 2s waits, then reports "the strand could NOT be removed after 3 attempts … remove it by hand" for a strand that is already gone.
   - `startupStrandRemoved` also stays false, so `startRetryable` refuses the `start_retries` relaunch that a died-before-first-input agent got before this change.
   - The pane and its window vanish too, so only an earlier `lastStartupCapture`, if any, survives for diagnosis.
   - `Runner.Open`, the precedent the review copied, has no startup teardown or retry, so the proposal does not carry over as it stands.
   - **Possible fix:** have the teardown treat an unknown strand as removed (`ErrUnknownStrand`, or `removeStrandIfPresent`), with a test row. Another option is to self-remove only after a successful start.

2. **LOW (F2).** Changed text: the `Spec.EndsWithSession` doc ("a resume drops a closed session") and `doc.go`'s "as an opened session does".
   - **Summary:** on codex, an agent that did not exit normally (reboot, killed pane) still comes back as a dead window on every resume.
   - `codexengine`'s `buildResumeCmd` is `sh.Exit()`, so the chained resume line is `exit; touch; remove; exit`. Nothing after the first `exit` runs, so no marker is written and no self-removal happens.
   - A session closed normally is covered, because the launch line wrote the marker. Only the interrupted case remains, and the old `open` path likely behaved the same. I did not check codex's `OpenLaunch`.

No other proposal carried out as proposed (F1, F3 to F10) broke in a case I could build.

**Tool calls**
- grep of `abandonStartup` and `removeStrandIfPresent`: the startup teardown body was not in my context.
- grep of reed's `RemoveStrand`: its handling of an unknown guid was not in my context (the output showed only return lines).
- read of `strand.go`'s `RemoveStrand` and `removeStrandLocked`: needed to confirm the `ErrUnknownStrand` path.
