Task: challenge the reviewer's proposals for F1 and F2 as the fixer carried them out.

Neither finding was fixed exactly as proposed, so I challenged the fixer's substituted change in each case.

- **F1 (own change: test in `internal/boardcli/verbs_test.go`, not in `internal/boardengine`):** I found no case where this is wrong. The proposal's seam, `export_test.go` in `boardengine`, would have had to reach `resolveLookup`, which lives in `boardcli`. Running the real `remove` verb on the payload from `RemoveCommand` tests the parser end to end, and `get` then confirms the entry is gone. The gate passed with the new step registered in `TestCLI`. The three slugs are dashed names that `agentname.ValidateSlug` accepts, so the single-quote hole the review rejected stays out of reach.
- **F2 (the review's "delete the fallback branch" alternative):** the replacement `removal := err.(*fronttask.RemoveError)` in `removeFixTask` is the weakest point, but it holds. `fronttask.Remove` is a concrete function whose every non-nil return is a bare `&RemoveError{...}` (`fronttask.go:225,229,241,244`). The assertion therefore cannot fail today, and a violation would be a programmer error that should fail loudly. `errors` and `boardengine` are still used elsewhere in `green.go` (`errors.Is` at the pair-path check, `entryDone`), so the imports still compile. No test or other text depended on the deleted fallback line: the review says no row reached it, and the gate passed.

Findings: no findings.

Further tool calls: none. My context already held every changed file and the gate result.
