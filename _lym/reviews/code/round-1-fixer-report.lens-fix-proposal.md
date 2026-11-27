```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that were carried out (F2, F3, F4).

1. **LOW**: F2's "The loop re-steps it once" in the `AnomalyTransientStop` doc comment (`internal/loomengine/anomaly.go`), the `internal/loomcli/doc.go` paragraph and `friction-template-reflection.md`.
   - **The claim is unconditional, but the re-step is not.** A `transient-stop` note is also written for a halt that no loop re-steps. `internal/loomcli/arm.go:569` calls `c.failedHalt(runErr)` on the `lym loom run` path. `runLoop` is entered only from the step verb's loop branch (`internal/shedverbs/step.go:361`), so a run or a bare step gets no automatic re-step. The reported incident was a bare `lym loom step`.
   - **"Once" is bounded differently in the loop.** `internal/shedverbs/loop.go` re-steps only while `restepped` is false, and only an ok result resets it. A second transient halt after a re-step therefore ends the loop with no further re-step.
   - **Possible wording:** "A step loop re-steps it once" states the condition.
   - **"The loop" is undefined in these three texts.** The reflection agent in particular has no antecedent for it.

2. **F3: no finding in the code comment.** The weaker claim holds. Outside this change set, `_lym/landing/summary.md` still says the two times show "whether the lock was held that long or the host was suspended". That is the same overclaim, left as a twin of the corrected comment (**LOW**; the fixer may not be able to write it).

3. **F4: no finding.** The new row sets `buildSkew` and `transient` without `interrupted`, so swapping the two cases would return `transient-stop` and fail it.

Tool calls:
- One `grep` for the `failedHalt`, loop and `decideDrive` call sites. The context showed the loop body and `failedHalt`, but not which verbs reach them.
