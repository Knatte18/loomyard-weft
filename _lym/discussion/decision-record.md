## Goal

```markdown
Drift-Verify's verdict reaches neither the change summary nor the pull request, so the PR asks the operator to run by hand the compare Drift already ran.

## Seen

On the `cs` hub (2026-11-03), `pipeflow-reynolds-helper` (darn, light) ran Drift-Verify at 15:30:06: the compare ran in the anchor against the baseline captured on the branch point, exited 0 and printed no `drift:` line, and the row ended `done`.

- Darn-Write wrote `_lym/landing/summary.md` before Drift-Verify ran; the summary as rewritten at 15:33, after Drift-Verify, and published by Describe at 15:33:54 still ends:
  "To check by hand: Run the regression harness compare against the baseline captured before this change, and confirm it reports no drift. It was not run as part of writing this description."
- Pull request Knatte18/lym-csharp#1 carries that text as its body.
- The PR-Review agent told the operator: "the summary says the regression harness compare has not been run yet. Its job is to confirm no result changed."
  Its prompt (`.lym/shuttle/<run>/prompt.md`) holds no word of Drift.

## Expected

- Describe adds Drift's outcome to the summary and so to the pull request: the scope (`full` or the skips), the baseline's branch point, the compare's result, and each drifted test with its accept reason or the waive reason.
- The PR-Review prompt names that outcome, so the reviewer does not ask the operator to repeat it.
- A summary line that asks for a check Drift has made is dropped or rewritten from the verdict.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-03): Drift verdict appended in Go, not written by Describe

Decision: Keep the Go code that appends Drift's outcome to the summary as it stands, in place of having Describe write it into summary.md. No further rewrite of a summary line asking for a hand check is needed in code; the Describe stencil tells the writer that Drift already compared and not to ask for the compare by hand.

Rationale: A section appended by code is exact and cannot drift from the record, which an agent's prose can; a stale hand-check line when Describe is skipped is a corner case the appended verdict next to it already contradicts.

### Added after Discussion (parent, 2026-11-03): Drift verdict in the landing commit message is kept

Decision: Keep the verdict appended to the landing commit message, beyond the summary, pull request and PR-Review prompt the entry names.

Rationale: It costs nothing, and the verdict in main's history is where a later reader looks for it once the pull request is gone.

## Constraints

## Auto-mode assumptions

## Open risks

## Acceptance criteria
