```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked every refusal, gate failure, copy step and criterion the fix introduced for whether it can be reached and leads to progress.

**MEDIUM: the copy now runs after acceptance, so "a failed review exit archives that copy" can never happen.**
- **Changed text:** D4's copy bullet ("When the round's review is accepted, one step … copies a `done` report … The copy joins the outputs the review row archives on a failed exit"), and the D4 criterion "a `done` parent report is copied beside the review file once the round's review is accepted, and a failed review exit archives that copy".
- **Evidence:** In `internal/shedadapters/burlerreview.go:441-469`, `acceptedExit` writes the acceptance record first (`:442`). The only archive call in that function is at `:443`, when the acceptance write fails, and no copy exists yet at that point. After a successful write, nothing archives anything: the remaining exits are a cancel error (`:449-451`) or the Stuck hand-on (`:469`). The hand-on path (`:257-259`) archives nothing either.
- **Why it matters:** Before the fix, the copy ran when the table ended, inside burlerengine and before acceptance, so a failed acceptance write archived it. Moving the copy to "when the round's review is accepted" leaves no failed exit after the copy. The archive sentence is dead, and the criterion's second half cannot be tested.
- **Ambiguity:** If "accepted" means burlerengine's `acceptReview` (`internal/burlerengine/review.go:276`) rather than the shedadapters acceptance record, the archive does stay reachable. But the hand-on path keys on the shedadapters record (`readAcceptance`, `:253`), so the step's two triggers name two different moments. The text should pick one: copy before `writeAcceptance` so a failed write archives it, or drop the archive clause and its criterion half.

**LOW: when the parent cannot be resolved, the refusal has no orch to name.**
- **Changed text:** the D4 bullet "Every other refusal … names the way forward: … `lym loom decision add <slug> --by parent`, run by the run's parent orch, which the refusal names as the verb's guard resolves it", and the D4 criterion "each refusal naming `lym loom decision add` and the run's resolved parent orch".
- **Evidence:** The guard's resolution can fail. `internal/loomcli/decision.go:685-690` then falls back to "the run's parent could not be resolved (…); the operator's own unnamed shell may still run the verb", and `:697-698` covers a parent with no name ("the operator's to record").
- **Why it matters:** The new refusal text and criterion assume a named parent. They give no way forward for an unresolvable or unnamed parent, where the existing guard sends the call to the operator's unnamed shell.

**LOW: a review that fails the coverage gate halts the row, and the record names no way forward from there.**
- **Changed text:** D7, "The entry never passes on its cap, so a review that still leaves an id uncovered fails the gate, and the round has no accepted review."
- **Evidence:** A failed gate makes burlerengine return an error (`internal/burlerengine/review.go:271-272`). The review row maps that error to `failureExit` (`internal/shedadapters/burlerreview.go:346-347`), which archives the round's outputs and returns a hard error. It is not retried in the row's attempt loop, which retries only died or timeout outcomes (`:354-359`).
- **Why it matters:** "No accepted review" therefore means the row halts until a restart. The record does not say so, and it does not say that the gate's failure reason, naming the missing ids, is what the operator or the restarted chair acts on.

The other new ways forward are reachable and lead to progress:
- The `--ask <latest>` refusal, while the latest ask is `pending-post` or `open`.
- The hand-on copy of an uncopied `done` report after a crash between `writeAcceptance` and the copy.
- The `not-sent` round with the key off, reached through a restarted row.
- The await's state, reason and path.
- The gate's re-prompt naming the missing ids.

Tool calls:
- grep of `internal/shedadapters/burlerreview.go` for failure exits and acceptance writes: my context did not show what the row does after acceptance.
- Read of `internal/shedadapters/burlerreview.go:340-469`: my context lacked `acceptedExit` and the attempt loop's error mapping.
