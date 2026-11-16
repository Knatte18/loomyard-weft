---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: internal/beamcli/cli.go:81
    summary: The help text of lym beam pr open lists every refusal but leaves out the two new ones, so it disagrees with the package doc and role-beams.md
    fix: In prOpenCmd's Long text in internal/beamcli/cli.go, add two bullets to the refusal list that opens with "It runs anywhere in the hub and refuses", after the bullet on an origin not on GitHub. The first reads, in the list's quoting style, a beam whose merge task is not done, naming lym batten status of the task. The second reads, a beam that conflicts with its parent branch, naming lym beam merge-parent with the slug. Add one sentence after the list saying that both are checked before a push or a reopen, and that a beam only behind its parent still opens.
    origin: lens:code-affected-set
  - id: F2
    severity: LOW
    class: design
    location: internal/beamcli/pr_test.go:128
    summary: No TestPROpen row pins the refusal order judgeOpenable documents (merge task before conflict), or that a beam only behind its parent still opens
    fix: In TestPROpen in internal/beamcli/pr_test.go, add two rows. Row one sets both an open b-merge-in task and parentConflicts=true, and wants the error to name "lym batten status b-merge-in" and not "lym beam merge-parent". Row two sets parentUnmerged=true with parentConflicts false, scripts a Create answer, and checks that the pull request is created and recorded.
    origin: lens:code-tests
  - id: F3
    severity: LOW
    class: design
    location: internal/beamcli/pr.go:242
    summary: The error paths of judgeOpenable, a failed board read and a failed ParentConflicts read, have no test row, and the landWorld fakes cannot fail them
    fix: In internal/beamcli/land_test.go, add a tasksErr field and a parentConflictsErr field to landWorld and return them from the Tasks and ParentConflicts fakes in deps(). In TestPROpen in internal/beamcli/pr_test.go, add a row with parentConflictsErr set. It wants the refusal to name "read whether the beam conflicts with its parent" and "lym beam pr open b", with wantUntouched true.
    origin: lens:code-tests
---

### [MEDIUM] The `pr open` help omits the new refusals

**Location:** internal/beamcli/cli.go:81
**Issue:** The `Long` text of `prOpenCmd` lists the verb's refusals in full under "refuses, each naming its way forward". It does not name the open merge task or the parent conflict that `judgeOpenable` (`internal/beamcli/pr.go:241`) now refuses. The diff updates the package doc (`internal/beamcli/doc.go:161`) and `contracts/orchrole/role-beams.md:39` with these refusals, so `--help` is now the only description of the verb that disagrees with the code.

### [LOW] Refusal order and the behind-only pass are not pinned

**Location:** internal/beamcli/pr_test.go:128
**Issue:** `judgeOpenable`'s doc comment (`internal/beamcli/pr.go:237-240`) promises two things: the merge task refuses first, and a beam only behind its parent passes. `doc.go` and `summary.md` claim the same. No `TestPROpen` row sets both an open merge task and a conflict, and none sets `parentUnmerged` true while expecting the open to go ahead. Swapping the two checks, or gating on `ParentUnmerged`, would fail no test.

### [LOW] `judgeOpenable`'s failure paths are uncovered

**Location:** internal/beamcli/pr.go:242
**Issue:** If `Tasks` or `ParentConflicts` fails, for example on a failed fetch of the parent's upstream, the code returns a rerun refusal before anything is pushed. The fakes at `internal/beamcli/land_test.go:167` and `:207` never return an error, so no row reaches these paths. That leaves the refusal's wording and its no-side-effect promise unchecked.

## Focus verdicts

- **Every path to a ready or reopened pull request passes `judgeOpenable`.** Holds for `prOpen` with none recorded: the check runs after `requireOrigin` and before `PushBeam`, `FindOpen`, `SetDraft` and `Create`. It also holds for `openRecorded`'s closed case, where the check runs before `Reopen`. The recorded-open case answers `already` and is not checked. That case opens nothing, and the board entry asks only that opening be refused, so it is no defect. `prUpdate` (also reached from `lym beam describe`) still marks a draft ready unchecked. A beam's recorded pull request is a draft only if someone turned it into one by hand, since `pr open` creates non-drafts and marks a found draft ready behind the check. That is outside the entry's ask, so it is no finding.
- **`openMergeTask`.** It matches on kind task, `Project == slug`, a status that is not done, and `darnengine.IsMergeTaskSlug`. It is the same helper `merge-parent` and `ready` already use, and `mergeparent_test.go:61` pins its done case. It raises no false refusal from another beam or from a finished task.
- **`ConflictsWith`.** `MergeTree` gives `clean=false` with a nil error only on exit 1, which `ConflictsWith` reads as a conflict. Every other git failure comes back as an error and is kept apart from a conflict. The pair's code HEAD is the beam's head, since `fabricengine.Open(beam)` opens the beam's pair.
- **`beamParentTips` refactor.** `ParentUnmerged` keeps the same order of reads, the same fetch condition and the same error returns. Both `landDeps` constructions, `hubLandDeps` and `landWorld.deps`, fill `ParentConflicts`.
- **Tests.** `TestPROpen` covers the conflict refusal, the merge task refusal and the closed-reopen refusal, each with no side effect. `ConflictsWith` is driven over real git through `TestHubLandDeps_ReadyReads`, in both its clean and its conflict result. Remaining gaps are F2 and F3.
- **Docs.** The `doc.go` and `role-beams.md` wording matches the code. `cli.go` is the gap (F1).
- **Fasit.** The operator's decision binds only "not opened before its conflicts with the parent are fixed". The Idea gives a choice of refusing, running the merge-in first, or opening a draft, and the diff takes the refusal. A beam that conflicts with its parent is refused, and so is the reported case's merge-in member that is still landing. The commit message's claim, a check before a push or a reopen, holds against `pr.go:81` and `pr.go:140`. Every claim in `summary.md` matches the diff.

## Rejected

- code-correctness NIT, the `ConflictsWith` HEAD-resolve error wraps under "conflict test against <ref>": the wrapped `ResolveSHA` error names HEAD itself, and the prefix names the operation, the same shape its sibling `HeadHoldsCommitsNotIn` uses.
- code-correctness LOW, `ParentConflicts` discards the head `beamParentTips` read: the result is unaffected, and `ConflictsWith`'s general one-ref signature is what PATTERN-fabric-boundary asks of a fabric seam.
- code-tests LOW, no fabricengine test of `ConflictsWith`'s error branches: the unresolvable-ref and merge-tree-failure paths are plain wraps of `gitrepo` errors, and their callers pass resolved tip SHAs. The verb's two results are pinned over real git in `TestHubLandDeps_ReadyReads`.
- code-tests LOW, recorded-open answering `already` while conflicting is not pinned: that behaviour is unchanged by the diff and outside the entry's ask.
- code-tests NIT, `ParentConflicts` against the upstream tip after a fetch is not covered: the tip read is the shared `beamParentTips` reader, unchanged from `ParentUnmerged`'s earlier body, whose upstream path had the same coverage before the diff.
- Handler candidate, the Idea's "a member is still landing" read as any member: the operator's decision binds only conflicts. The Seen case is a merge-in member, which the diff refuses, and refusing on every landing member would block a beam whose members land continually.
