```yaml
lens: code-scope
outcome: reported
```

Task: the code-scope lens. I mapped each change in the diff to the board-entry goal point it builds.

**Decisions.** The answer key is the decision record alone. Its `## Goal` holds the board entry, and the record has no `Added after Discussion` entries.
- **G1 (#176):** accept the finished report, or say in the fixer prompt why the step runs again (a misnamed or rejected report versus new work).
- **G2 (#177, part 1):** scope the mutating-git guard to the target repository and its remotes, so git in a repository the fork created under `.scratch/` passes.
- **G3 (#177, part 2):** make the halt say when no target change was found.

**Changes, each with the decision it traces to.**
- **C1:** `cluster.go` push scoping. This covers `pushMutates`, `recordRemote`, `setRemote`, `leadsInto`, `isGitURL`, `namesPushRedirectKey`, `redirectsPush` in `parseGitInvocation`, and the push, remote and redirect-key option tables. **Traces to G2.**
- **C2:** `cluster.go` clone recording: `recordClone`, `cloneDirectoryName` and the clone option tables. A clone is a repository the fork created, so this **traces to G2.**
- **C3:** `cluster.go` refactors: `forkRepositories` becomes a struct, `firstOperand` becomes `gitOperands`, `contain` becomes `rootOf`/`rootContaining`, and `record` returns the root. **Forced by C1/C2.**
- **C4:** `forkViolationError`, `fixViolationWayForward`, and the `auditFixRound` wrap that uses it. **Traces to G3.**
- **C5:** `firstMutatingGitCommand` becomes `mutatingGitCommands`, which returns every violation with `pushFrom`. **Forced by C4,** whose no-target-change call needs every violation.
- **C6:** the git-violation message gains "outside a repository it created" (`cluster.go:754`). **Forced by C1/C2,** which changed the rule the message states.
- **C7:** `cluster_test.go` rows and the way-forward assertion. **Forced by C1/C2/C4/C5.**
- **C8:** `burlerengine/doc.go` paragraphs on push scoping and the way forward. **Forced by C1/C4.**
- **C9:** `FixRerun`, `RunOpts.Rerun` (`profile.go`), `fixWork`/`fixRerunStatement` (`prompt.go`), the `fix.go` wiring, and the `doc.go` rerun paragraph. **Traces to G1** (the "state in the fixer prompt why" branch).
- **C10:** `recordRerun`, `recordedRerun` and the `archiveReport` reason in `shedadapters/burlerfix.go`, plus the `fixer_rerun` record in `roundsnapshot.go`. **Traces to G1.**
- **C11:** `archiveStaleOutput` split out of `archiveStaleOutputs` in `archive.go`. **Forced by C10,** which needs the archived path.
- **C12:** the `burlerfix_test.go`, `prompt_test.go`, `fix_test.go` and `template_test.go` edits, and the `shedadapters/doc.go` paragraph. **Forced by C9/C10.**

**Decisions, each with the changes that build it.**
- **G1:** C9, C10, C11, C12. The "accept the finished report" branch is not taken. The goal offers "or", and the summary says why it was not taken.
- **G2:** C1, C2, C3, C6, C7, C8.
- **G3:** C4, C5, C7, C8. The only halt that carried the revert text was `auditFixRound` (`cluster.go:774`), and it is the one the diff changes.

**Findings.** None. Every change either traces to G1, G2 or G3 or is forced by a traced change, and every goal point is built by at least one change. No commit in the range carries a `Pattern-Departure:` trailer.

**Tool calls.**
1. Grepped `internal` for "revert what the fork" and "no target change" in production code, to confirm whether any halt besides `auditFixRound` carries the revert way forward that G3 covers. My context held only `cluster.go`'s copy and had no search across the repository.
