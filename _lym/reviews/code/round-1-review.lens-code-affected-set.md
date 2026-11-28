```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: what the diff's changed names, messages, briefs and docs force outside its own lines.

**Set per change**

1. **Rename `fixViolationWayForward` to `violationWayForward`.** There are no other references anywhere in the repository. Covered.
2. **`pushRedirected` became `pushRedirect`, with `redirectPushes` added.** The only references are in `cluster.go`, and every writer goes through `redirectPushes`. Covered.
3. **`runsMutatingGit`, `gitInvocationMutates` and `pushMutates` now return `(pushFrom, rule string)`.** Their only caller is `mutatingGitCommands`, and its doc was updated. The doc on `mutatingGitCommand`, which the diff left unedited, is now wrong (F1).
4. **The violation message is now "…counted as <rule> — … may change a repository it did not create".**
   - No other Go, spec, doc or tool file holds the old text.
   - The tests at `cluster_test.go:116,138,309,324` match the new text.
   - The shedadapters callers classify only `ErrClusterForksMissing` and `ErrHalfNotStopped`; the violation stays unclassified and fails through `failureExit`, as at the base. Covered.
5. **`auditClusterRound` now appends the way forward.**
   - Its caller is `acceptReview` (`handoff.go:44`), which passes the error through unchanged.
   - `fmt.Errorf` keeps `%w`, so `errors.Is` and `errors.As` still reach the violation.
   - The package doc now says both paths end with a way forward. The fixer section at `doc.go:105-107` is unchanged and still true.
   - The doc comments on the test constants that the review-path rows now use were left stale (F3).
6. **Brief rewrite in `burler-cluster-fan.md` and `burler-fix-self-check.md`.**
   - No test asserts the brief prose (searched `_test.go` files).
   - The renderers at `prompt.go:594` and `prompt.go:652` read the stencils by name, so the brief's shape is unaffected.
   - The sibling brief `burler-assist-cluster-fan.md:21` is unchanged; the change description says no fork audit covers it, and I did not verify that.
   - `twill-directive.md:43` is a different audit and out of scope.
   - The rewrite names no denied recovery form from PATTERN-refusal-way-forward.
   - The package doc's own "Fork discipline" lead now contradicts the briefs (F2).
7. **`doc.go` fork-discipline paragraph.** The added lines agree with the code. Lines 254-256, left untouched, now disagree with the new briefs (F2).
8. **`docs/overview.md:1685`** ("forks of a code perch reach the diff through read-only git") is still true of the target. Covered.
9. **Repository rules.**
   - PATTERN-refusal-way-forward binds webster, shed and loom. `burlerengine` is not bound, but its error reaches the shed halt, and the halt now carries a `way forward:` clause.
   - The rule texts are named constants, as PATTERN-code-conventions asks.
   - No declared PATTERN changes, and no `Pattern-Departure:` trailer.
   - The `Root-Cause:` trailer on `cec5ca742` names the defect the diff removes. Covered.

**Findings**

- **F1 — LOW, consistency.** `internal/burlerengine/cluster.go:127-128#mutatingGitCommand`.
  - The type doc still says "The invocation runs outside the repositories the fork created." A counted push after `ln`, a redirect key or an unknown-directory remote change runs from a created repository. The new `pushFrom` field documents exactly that case, at `cluster.go:134`.
  - The `mutatingGitCommands` doc at `cluster.go:140` ("runs a state-mutating git subcommand on a repository the fork did not create") also disagrees with the new message wording "may change a repository it did not create" (`cluster.go:927`).
  - Fix: reword both to "may change state in a repository the fork did not create".
- **F2 — LOW, consistency.** `internal/burlerengine/doc.go:253-256`.
  - The fork-discipline lead still says "no write of any file; no git command that changes the repository, index, refs or working tree".
  - Both briefs now permit state changes in a throwaway repository and forbid only writes to the target and write/edit tool calls (`burler-cluster-fan.md:22`, `burler-fix-self-check.md:45-48`). The same doc paragraph's own allowance at `doc.go:266-267` says the same.
  - Fix: restate the lead as "no write or edit tool call; no state-changing git command outside a repository the fork created".
- **F3 — LOW, consistency.** `internal/burlerengine/cluster_test.go:29,32`.
  - The doc comments on `revertWayForward` and `untouchedPushWayForward` still call them "the fix-round halt's way forward". The diff now asserts `revertWayForward` on a review-path row (`cluster_test.go:310`), and the review path produces the same text.
  - Fix: drop "fix-round" from both.

Outside my lens: the change description's claim "Every row's way forward is checked on both the review and the fix paths" does not hold. Each row runs exactly one path, chosen by `tt.fix` (`cluster_test.go:355-363`).

Tool calls:
1. Repository-wide grep for the old names and message text, to find readers outside the diff; the manifest held only `cluster.go` and its tests.
2. Grep for sibling briefs and for code that reads the two stencils, to find stencil readers outside the diff.
3. Read `doc.go:98-112`, the sibling briefs' read-only lines and specs mentioning forks, to check the fixer-fan doc and other fork briefs the manifest lacked.
4. Grep `docs/overview.md` for audit or throwaway mentions, to find a repository doc describing fork discipline.
5. Grep tests for assertions on the brief prose, plus the diff stat, to rule out stencil-prose tests and confirm the changed file set.
