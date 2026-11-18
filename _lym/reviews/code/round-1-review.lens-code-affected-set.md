```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the affected set and whether the diff covers it**

1. **`DeliverPendingPrompt` split into `typePendingPrompt`; `promptSentAt` now set on delivery.**
   - Callers: `deliverPromptAtHold` (hold.go:146), `deliverPromptOnAttach` (hold.go:170), and `burlerengine.ResumeFix` through `promptDeliverer` (internal/burlerengine/fix.go:288). Their contracts are unchanged. Covered.
   - doc.go "# Pending prompt" (doc.go:262-284) gains the retype paragraph. Covered.
   - `Run.promptDelivered` field comment (run.go:326): not updated, see F2.
2. **New `retypeLostPrompt` on every `Wait` tick.**
   - `Wait` loop (wait.go:313), the hold.go file header (diff line 119) and doc.go (diff lines 91-96). Covered.
   - PATTERN-reed-panes "What each caller does with `ErrPaused`" says "the deferred prompt wait out the pause and resend". The retype goes through `sendThroughPause`, so the entry still holds. Covered.
   - PATTERN-daemon-wakeups: the change adds no new wait loop and no sub-second literal. Covered.
   - PATTERN-shuttle-provider-seam completion signal: the change adds no negative outcome. Covered.
3. **`sendStartPrompt` busy retry and `startPromptBusyRetries`.**
   - Caller: `loadSkillsThenPrompt` (run.go:870). Its doc comment (run.go:823-824) and doc.go (diff lines 104-107) are updated. Covered.
   - PATTERN-reed-panes: "A write the guard holds fails wrapping `ErrSessionBusy` and writes nothing, and the caller retries later." Consistent. Covered.
4. **`keepFailureCapture` changes what `startup-capture.txt` holds after a prompt delivery failure.**
   - `Run.lastStartupCapture` comment (run.go:364-369) updated. Covered.
   - `startupCaptureFileName` comment (run.go:492-494) says "the last successful pane capture on a not-ready teardown". It stays true for the not-ready case. Covered.
   - docs/reference/claude-trust-dialog-repro.md:42 describes only the not-ready trust-dialog case. Unaffected.
   - doc.go:931 is unaffected.
   - The `abandonStartup` doc (wait.go:805) is not updated, see F1.
   - Tests: startup_test.go:349-447 and :1179 cover not-ready teardowns, which the change does not touch. The new assertion is at :2359. Covered.
5. **`startSendEngine.idleWhen` test fake and the new test rows.** Fixture only, used only by the touched test. Covered.
6. **Commit messages and departures.** Neither commit carries a `Pattern-Departure:` trailer, and the diff departs from no PATTERN entry that would need one. The rendered directive declares no PATTERN changes, so there is nothing to check under "Declared PATTERN changes landed". No stencil is changed.

**Findings**

- **F1.** Severity LOW, class `consistency`.
  - Location: internal/shuttleengine/wait.go:805, the `abandonStartup` doc comment.
  - Summary: the comment still says it "saves any capture that checkLivenessTick recorded to startupCaptureFileName". After a failed prompt delivery, the saved capture is now the one `keepFailureCapture` took (run.go:906-913), not a `checkLivenessTick` capture. The comment is stale.
  - Evidence: wait.go:805 and run.go:874 and 906-913.
- **F2.** Severity NIT, class `consistency`.
  - Location: internal/shuttleengine/run.go:326, the `promptDelivered` field comment.
  - Summary: the comment says the latch exists "so no later call retypes it". `retypeLostPrompt` (hold.go:205-206) now types the line again through `typePendingPrompt` after a delivery that set the latch. The comment should limit "no later call" to `DeliverPendingPrompt`.
  - Evidence: run.go:326, run.go:952-955 and hold.go:198-206.

**Tool calls**

1. Grepped for `startup-capture`, `startupCaptureFileName` and the pending-prompt docs, and read doc.go:250-282. The manifest held no readers of the capture file or the doc.go pending-prompt section.
2. Read wait.go:798-822, docs/reference/claude-trust-dialog-repro.md:36-46 and run.go:488-496. These are the docs describing the capture file, which the first grep found and the manifest lacked.
