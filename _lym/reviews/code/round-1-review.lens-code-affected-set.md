```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what `readinessState` and its doc and fake-engine edits force outside the touched lines.

**Per change: set found, covered or not**

1. `sessionReadiness` now folds through `readinessState`. Its callers:
   - `busyReading` (`send.go:239`), which serves the verified send's idle wait. Covered, the intended path.
   - `Runner.SessionIdle` (`session.go:266`). It reads the same fallback state as the send path. Covered, and the summary and `doc.go:339` state it.
   - Nothing else calls `sessionReadiness`. The doc comment above `sessionReadiness` names `readinessState`. Covered.

2. Readers of the offset-bounded fold that stay on `readRunSessionState`:
   - `conditionedReading` (`conditionedsend.go:66`)
   - `continueAPIErrorStall` (`hold.go:174`)
   - `agentwait.go:59`
   - `Runner.SessionState` (`session.go:156`)
   - `ReadSessionStates` (`sessionread.go:66`)
   - Each judges "what has this run done since the prompt", so it should stay bounded, and `Wait`'s shadow and cursors keep `PromptOffset` (`wait.go:288-293`). The `doc.go:524` sentence "Both read the signals past the prompt offset" covers `ReadSessionStates` and `Runner.SessionState`, which are unchanged. Covered.
   - `activity.go:243` already falls back to the whole file when nothing lies past the offset, so the fallback has precedent.

3. `doc.go:339-340` addition. It sits in the readiness and session-state folding paragraph, next to the "unknown state falls back to the pane" sentence. It matches the code on the no-signal cause and the non-zero offset. No other `doc.go`, `PATTERN.md` or `pattern/` entry describes the readiness fold, and the grep found no contradicting text. Covered.

4. `fakes_test.go` `hookStateSkillEngine.IdleSession`. Its only users are `startup_test.go:1455`, which builds the fake for the hook-state start fixture, and the `fakes_test.go` fixtures. The doc comment was updated with the change. Covered, though the weakening question belongs to the code-tests lens.

5. PATTERN entries: none of the listed entries touch this change. The commit carries no `Pattern-Departure:` trailer and needs none. No declared PATTERN changes exist to land.

6. Stencils and specs: the grep found no stencil text about the readiness fold. Nothing to update.

**Findings:** none.

**Tool calls (1)**
- `grep` of `doc.go` and the package sources for text describing the offset-bounded fold, to check for other docs the change contradicts. My context held only the `doc.go` diff hunk.
