---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/landingshed/doc.go:44 and internal/shedrecipe (darn recipe PR-Review and Describe rows)
    summary: The answer key asks for no PR-Review row, but the darn recipe keeps the row, which passes at once, and Describe still runs; the change description names this as a design decision, not a recorded correction.
    fix: Leave the code in place and defer this finding to the run's parent, who records the deviation with lym loom decision add or refuses it; no code change is the fixer's to make.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-correctness
    location: internal/beamcli/mergeparent.go:126
    summary: The merge task body renders "Filed by`lym beam merge-parent X`" with no space before the backtick, a regression from the original text, and no test pins the body line.
    fix: In mergeParentBody restore the space so the line reads Filed by followed by a space and then the backtick-quoted command, as before the diff.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/boardengine/task.go:201-202
    summary: The normalizeTask doc comment still says a PullRequest other than empty, on and off is refused, but merge-in is now accepted.
    fix: In the normalizeTask doc comment change the sentence to say a PullRequest other than empty, on, off and PullRequestMergeIn is refused for a task and a project alike.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_pr_test.go:229
    summary: The row named AMergeInEntryLandsByATrueMergeWithNoPullRequestAndNoRuleReadUnderARequiredParent claims no rule read, but no assertion backs it, so the TrueMerge short-circuit in resolveWithParentRule can be deleted with every test green.
    fix: In that row assert that the GitHub rule-read seam is never called for the merge-in entry, or drop NoRuleRead from the row name.
---

### [MEDIUM] PR-Review row and Describe still run for a merge-in task

**Location:** internal/landingshed/doc.go:44 and the darn recipe's PR-Review and Describe rows.
**Issue:** The answer key (issue #65) wants the merge task to land "with no PR-Review row and no GitHub pull request".
The diff changes only the landing resolver, so the row stays in the recipe and passes at once on the resolver's no, and Describe still runs when the Code-Review perch is on.
No GitHub pull request is opened, so the observable result holds, but the row is not gone.
`_lym/landing/summary.md` states this as a design decision, with recipes having no per-entry rows as the reason, so it is a deviation from the amended entry that the record does not carry.
It is not fixed by revert: the parent records or refuses it.

### [LOW] Missing space in the merge task body

**Location:** internal/beamcli/mergeparent.go:126.
**Issue:** The diff turned ``Filed by `lym beam merge-parent X` `` into ``Filed by`lym beam merge-parent X` ``, which renders badly in the board body the writer and the perch read.
`TestMergeParent` checks only two other substrings, so the regression is unpinned.

### [LOW] normalizeTask doc comment is stale

**Location:** internal/boardengine/task.go:201-202.
**Issue:** The diff updated the `Task` field comment and the refusal text to accept `merge-in`, but left the `normalizeTask` doc sentence saying only empty, on and off are accepted.

### [LOW] A test row claims a rule-read check it does not make

**Location:** internal/landingshed/finalize_pr_test.go:229.
**Issue:** The row's name promises that no GitHub rule is read.
Nothing asserts it, and a merge-in answer is the same with or without the read, so removing `decision.TrueMerge` from the early return in `resolveWithParentRule` (internal/landingshed/pullrequestmode.go:159) passes every test.

## Rejected

- Lens code-tests, no PR-Review row for merge-in: PR-Review is unchanged by the diff, its `!decision.Open` path is the one the existing off-mode rows cover, and the resolver row pins merge-in's answer, so a new row would duplicate coverage.
- Lens code-tests, no test that the parent's commit becomes an ancestor: the Finalize row runs the parent-side merge call and asserts it is not squashed, and non-squash ancestry is fabricengine's existing contract, so the diff adds no uncovered behavior.
- Lens code-affected-set, a leftover open pull request record on a merge-in entry still squash-merges: Finalize's early return for an open recorded pull request is existing behavior, and PR-Review closes any record for a merge-in entry before Finalize runs, so the path is unreachable in practice.
- Lens code-correctness, `merge <branch> --continue` passes the guard: the `merge` command's `Args` check refuses a positional argument alongside `--continue` or `--abort` (internal/fabriccli/merge_verbs.go:174-182), and cobra validates arguments before the guard's pre-run.
- Lens code-affected-set, the `run_settings` help line was shortened: the commit that did so was needed to bring the board upsert help under the Long cap, so it is not out of scope.
- Lens code-affected-set, the end-of-line comment on the `seam_enforcement_test.go` allow-list entry: its neighbours use the same form, so the diff follows the surrounding convention.
