```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-ran the set searches behind the review findings to check the fix covers every member.

Findings left unchecked: none.

1. **LOW: the F4 claim has a sibling the fix left alone.** The changed text is `_lym/landing/summary.md`, Design decisions: "except when the viewer sits on a status strand window the restore replaces, where reed moves the view to the replacement's window".
   - The same behaviour is described without that exception in `internal/loomcli/resume.go:7` ("never switches a tmux client") and in the `resumeCmd` Long ("It never spawns an LLM driver, adds any other strand or switches a tmux client").
   - `ReplaceStrand` calls `selectStrandWindowLocked` when the replace ends the window the viewer is on (`internal/reedengine/replace.go`, `endedCurrent && replacingWithOverview`), so an attached viewer's view does move on resume.
   - The sentences are literally true if "switches a tmux client" means only tmux `switch-client`. A reader would likely take them to mean the view stays put, which the corrected summary now says is not always so.

No other missed members:
- **F1:** `start.go` and the `wiring.go` closure are the `ensureStatusStrand` callers the earlier grep showed, and both handle the new error. The `status_pane` claims in `doc.go`, the Long and the summary now hold for a failed replace.
- **F2:** the new smoke assertion covers the real restore closure. The parked-driver call site is covered by the counting fake in `TestResumeVerb`.
- **F3:** I found no remaining "`start` only" or "next start" wording about the status strand.
- **F5:** I found no other "without starting anything" description of `lym loom resume`.

Tool calls (5):
1. grep for "start only", "next start" and "this run" wording across `internal` and `docs`: my context held only a narrower symbol grep.
2. grep for "starts nothing", "never switches" and resume/pane claims: my context had searched only the exact Short phrases.
3. grep for `start` and status strand, pane or sidecar co-mentions: sibling packages and docs were not in context.
4. grep for `lym loom resume` descriptions that restrict what it does: context held only one matching line of `docs/overview.md`.
5. Read the resume passage of `docs/overview.md`: its wrapped lines were not in context.
