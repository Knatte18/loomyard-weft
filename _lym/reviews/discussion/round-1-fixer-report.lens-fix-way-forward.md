```yaml
lens: fix-way-forward
outcome: reported
```

Lens fix-way-forward: I checked whether each refusal, halt, next step and acceptance case this fix added can be reached and leads to progress.

1. **MEDIUM.** The new D6 entry bullet ("Any other flag is judged as a wake judges it below, once the session is ensured") can halt a run in a loop that never reaches the step it should name.
   - **The case:** a flag carries the current stamp but the record is damaged.
     The entry check passes only "with a clean checkout and an intact record", so the row goes on to ensure a session first.
     If the recorded session can no longer be resumed, D5 makes that "Stuck with Halt naming `lym loom resume`".
   - **Why it loops:** every resume repeats the same path, so the halt never reaches the wake's "Stuck with Halt naming its restore steps".
     The named way forward, `lym loom resume`, therefore never makes progress.
     The new Bound sentence promises that a resume ends the row "even when the recorded session can no longer be resumed", but that holds only for an intact record.
   - **Evidence:** the D6 entry bullet's ordering against D5's start-failure bullet and the wake's damaged-record sub-bullet.
     A dirty checkout on entry has a similar but milder problem: the wake path "first stops the session … resumes the session" would stop a session that was just ensured and start it again.
   - **Fix:** run the record check, and the stuck-with-restore-steps halt, before the session is ensured.

2. **LOW.** The new D5 steer text, "lym pushes the branch and opens any pull request only after `sampler-done` ends the row", says what lym does but no longer gives the refused agent a step to take.
   - **Evidence:** the steer it replaces in `internal/shellguard/rows.go:100` ends with "commit your change and tell the operator".
     `PATTERN-refusal-way-forward` says a refusal message names the way forward.
   - **Fix:** add a step such as "commit your change and run `lym loom sampler-done` when the operator says the work is done".

Tool calls:
- A grep of `docs` for `PATTERN-refusal-way-forward`, to confirm the pattern's wording; my context did not hold the pattern text.
